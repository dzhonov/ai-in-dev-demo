## JUNIT 4 MessageService

### Запрос:

Implement unit tests for the class below, provide 100% coverage, use junit, write code:

```java
package com.testing.unittest;

import com.letscode.sarafan.domain.Message;
import com.letscode.sarafan.domain.User;
import com.letscode.sarafan.domain.UserSubscription;
import com.letscode.sarafan.domain.Views;
import com.letscode.sarafan.dto.EventType;
import com.letscode.sarafan.dto.MessagePageDto;
import com.letscode.sarafan.dto.ConfigDTO;
import com.letscode.sarafan.dto.ObjectType;
import com.letscode.sarafan.service.MessageRepo;
import com.letscode.sarafan.service.UserSubscriptionRepo;
import com.letscode.sarafan.service.WsSender;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.time.LocalDateTime;
import java.util.List;
import java.util.function.BiConsumer;
import java.util.regex.Matcher;
import java.util.regex.Pattern;
import java.util.stream.Collectors;

@Service
public class MessageService {
    private static String URL_PATTERN = "https?:\\/\\/?[\\w\\d\\._\\-%\\/\\?=&#]+";
    private static String IMAGE_PATTERN = "\\.(jpeg|jpg|gif|png)$";

    private static Pattern URL_REGEX = Pattern.compile(URL_PATTERN, Pattern.CASE_INSENSITIVE);
    private static Pattern IMG_REGEX = Pattern.compile(IMAGE_PATTERN, Pattern.CASE_INSENSITIVE);

    private final MessageRepo messageRepo;
    private final UserSubscriptionRepo userSubscriptionRepo;
    private final BiConsumer<EventType, Message> wsSender;

    @Autowired
    public MessageService(
            MessageRepo messageRepo,
            UserSubscriptionRepo userSubscriptionRepo,
            WsSender wsSender
    ) {
        this.messageRepo = messageRepo;
        this.userSubscriptionRepo = userSubscriptionRepo;
        this.wsSender = wsSender.getSender(ObjectType.MESSAGE, Views.FullMessage.class);
    }

    public void delete(Message message) {
        messageRepo.delete(message);
        wsSender.accept(EventType.REMOVE, message);
    }

    public Message update(Message messageFromDb, Message message) throws IOException {
        messageFromDb.setText(message.getText());
        fillConfig(messageFromDb);
        Message updatedMessage = messageRepo.save(messageFromDb);

        wsSender.accept(EventType.UPDATE, updatedMessage);

        return updatedMessage;
    }

    public Message create(Message message, User user) throws IOException {
        message.setCreationDate(LocalDateTime.now());
        fillConfig(message);
        message.setAuthor(user);
        Message updatedMessage = messageRepo.save(message);

        wsSender.accept(EventType.CREATE, updatedMessage);

        return updatedMessage;
    }

    public MessagePageDto findForUser(Pageable pageable, User user) {
        List<User> channels = userSubscriptionRepo.findBySubscriber(user)
                .stream()
                .filter(UserSubscription::isActive)
                .map(UserSubscription::getChannel)
                .collect(Collectors.toList());

        channels.add(user);

        Page<Message> page = messageRepo.findByAuthorIn(channels, pageable);

        return new MessagePageDto(
                page.getContent(),
                pageable.getPageNumber(),
                page.getTotalPages()
        );
    }

    private void fillConfig(Message message) throws IOException {
        String text = message.getText();
        Matcher matcher = URL_REGEX.matcher(text);

        if (matcher.find()) {
            String url = text.substring(matcher.start(), matcher.end());

            matcher = IMG_REGEX.matcher(url);

            message.setLink(url);

            if (matcher.find()) {
                message.setLinkCover(url);
            } else if (!url.contains("А")) {
                ConfigDTO config = getConfig(url);

                message.setLinkCover(config.getCover());
                message.setLinkTitle(config.getTitle());
                message.setLinkDescription(config.getDescription());
            }
        }
    }

    private ConfigDTO getConfig(String url) throws IOException {
        Document doc = Jsoup.connect(url).get();

        Elements title = doc.select("[property$=title]");
        Elements description = doc.select("[property$=description]");
        Elements cover = doc.select("[property$=image]");

        return new ConfigDTO(
                getContent(title.first()),
                getContent(description.first()),
                getContent(cover.first())
        );
    }

    private String getContent(Element element) {
        return element == null ? "" : element.attr("content");
    }


}
```

---

### Ответ:


```java
import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

import com.testing.unittest.MessageService;
import com.letscode.sarafan.domain.Message;
import com.letscode.sarafan.domain.User;
import com.letscode.sarafan.domain.UserSubscription;
import com.letscode.sarafan.dto.EventType;
import com.letscode.sarafan.dto.MessagePageDto;
import com.letscode.sarafan.dto.ConfigDTO;
import com.letscode.sarafan.service.MessageRepo;
import com.letscode.sarafan.service.UserSubscriptionRepo;
import com.letscode.sarafan.service.WsSender;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;

import java.io.IOException;
import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.List;
import java.util.function.BiConsumer;

@ExtendWith(MockitoExtension.class)
class MessageServiceTest {
    @Mock
    private MessageRepo messageRepo;

    @Mock
    private UserSubscriptionRepo userSubscriptionRepo;

    @Mock
    private WsSender wsSender;

    @InjectMocks
    private MessageService messageService;

    @Test
    void testDelete() {
        Message message = new Message();
        doNothing().when(messageRepo).delete(message);

        messageService.delete(message);

        verify(messageRepo, times(1)).delete(message);
        verify(wsSender, times(1)).accept(EventType.REMOVE, message);
    }

    @Test
    void testUpdate() throws IOException {
        Message messageFromDb = new Message();
        Message message = new Message();
        when(messageRepo.save(messageFromDb)).thenReturn(messageFromDb);
        BiConsumer<EventType, Message> wsSender = (eventType, updatedMessage) -> {
            assertEquals(EventType.UPDATE, eventType);
            assertEquals(messageFromDb, updatedMessage);
        };
        messageService.setWsSender(wsSender);

        Message updated = messageService.update(messageFromDb, message);

        assertEquals(messageFromDb, updated);
    }

    @Test
    void testCreate() throws IOException {
        User user = new User();
        Message message = new Message();
        Message updatedMessage = new Message();
        when(messageRepo.save(message)).thenReturn(updatedMessage);
        BiConsumer<EventType, Message> wsSender = (eventType, createdMessage) -> {
            assertEquals(EventType.CREATE, eventType);
            assertEquals(updatedMessage, createdMessage);
        };
        messageService.setWsSender(wsSender);

        Message created = messageService.create(message, user);

        assertEquals(updatedMessage, created);
    }

    @Test
    void testFindForUser() {
        User user = new User();
        Pageable pageable = mock(Pageable.class);
        Page<Message> page = mock(Page.class);
        List<Message> messages = new ArrayList<>();
        messages.add(new Message());
        when(userSubscriptionRepo.findBySubscriber(user)).thenReturn(new ArrayList<>());
        when(messageRepo.findByAuthorIn(anyList(), eq(pageable))).thenReturn(page);
        when(page.getContent()).thenReturn(messages);
        when(page.getPageNumber()).thenReturn(0);
        when(page.getTotalPages()).thenReturn(1);

        MessagePageDto messagePageDto = messageService.findForUser(pageable, user);

        assertEquals(messages, messagePageDto.getMessages());
        assertEquals(0, messagePageDto.getPageNumber());
        assertEquals(1, messagePageDto.getTotalPages());
    }
}
```