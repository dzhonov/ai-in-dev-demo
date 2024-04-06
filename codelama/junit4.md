## JUNIT 4 MessageService

### Запрос:

Implement unit tests for the class below, provide 100% coverage, use junit, write code:

```java

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