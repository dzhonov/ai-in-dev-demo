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

To implement unit tests for the MessageService class, you can use JUnit and mockito. Here's an example of how you can do it:

```java
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.when;

public class MessageServiceTest {
    private MessageRepo messageRepo = Mockito.mock(MessageRepo.class);
    private UserSubscriptionRepo userSubscriptionRepo = Mockito.mock(UserSubscriptionRepo.class);
    private WsSender wsSender = Mockito.mock(WsSender.class);
    @Test
    public void testDelete() {
        Message message = new Message();
        MessageService messageService = new MessageService(messageRepo, userSubscriptionRepo, wsSender);
        messageService.delete(message);
        Mockito.verify(messageRepo).delete(message);
        Mockito.verify(wsSender).accept(EventType.REMOVE, message);
    }
    @Test
    public void testUpdate() throws IOException {
        Message messageFromDb = new Message();
        Message message = new Message();
        MessageService messageService = new MessageService(messageRepo, userSubscriptionRepo, wsSender);
        messageService.update(messageFromDb, message);
        Mockito.verify(messageRepo).save(messageFromDb);
        Mockito.verify(wsSender).accept(EventType.UPDATE, message);
    }
    @Test
    public void testCreate() throws IOException {
        Message message = new Message();
        User user = new User();
        MessageService messageService = new MessageService(messageRepo, userSubscriptionRepo, wsSender);
        messageService.create(message, user);
        Mockito.verify(messageRepo).save(message);
        Mockito.verify(wsSender).accept(EventType.CREATE, message);
    }
    @Test
    public void testFindForUser() {
        Pageable pageable = Mockito.mock(Pageable.class);
        User user = new User();
        MessageService messageService = new MessageService(messageRepo, userSubscriptionRepo, wsSender);
        MessagePageDto messagePageDto = messageService.findForUser(pageable, user);
        assertEquals(messagePageDto.getContent(), messageRepo.findByAuthorIn(userSubscriptionRepo.findBySubscriber(user).stream().filter(UserSubscription::isActive).map(UserSubscription::getChannel).collect(Collectors.toList()), pageable).getContent());
        assertEquals(messagePageDto.getPageNumber(), pageable.getPageNumber());
        assertEquals(messagePageDto.getTotalPages(), messageRepo.findByAuthorIn(userSubscriptionRepo.findBySubscriber(user).stream().filter(UserSubscription::isActive).map(UserSubscription::getChannel).collect(Collectors.toList()), pageable).getTotalPages());
    }
}
```
In this example, we're using JUnit and Mockito to test the `MessageService` class. We're mocking the `MessageRepo`, `UserSubscriptionRepo`, and `WsSender` classes, and then using them to test the `delete`, `update`, `create`, and `findForUser` methods.

We're also using the `assertEquals` method to verify that the `MessagePageDto` object returned by the `findForUser` method has the correct content, page number, and total pages.

Note that this is just one way to test the `MessageService` class, and you may need to modify it to fit your specific needs.