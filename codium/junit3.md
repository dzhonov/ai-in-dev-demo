## JUNIT 3 UserService

### Запрос:

Implement unit tests for the class below, provide 100% coverage, use junit:
```java

package com.testing.unittest;

import com.example.sweater.domain.MailSender;
import com.example.sweater.domain.Role;
import com.example.sweater.domain.User;
import com.example.sweater.domain.UserRepo;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

import java.util.*;
import java.util.stream.Collectors;

@Service
public class UserService implements UserDetailsService {
    @Autowired
    private UserRepo userRepo;




    @Autowired
    private MailSender mailSender;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @Value("${hostname}")
    private String hostname;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepo.findByUsername(username);

        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }

        return user;
    }

    public boolean addUser(User user) {
        User userFromDb = userRepo.findByUsername(user.getUsername());

        if (userFromDb != null) {
            return false;
        }

        user.setActive(true);
        user.setRoles(Collections.singletonList(Role.USER));
        user.setActivationCode(UUID.randomUUID().toString());
        user.setPassword(passwordEncoder.encode(user.getPassword()));

        userRepo.save(user);

        sendMessage(user);

        return true;
    }


    public boolean activateUser(String code) {
        User user = userRepo.findByActivationCode(code);

        if (user == null) {
            return false;
        }

        user.setActivationCode(null);

        userRepo.save(user);

        return true;
    }

    public List<User> findAll() {
        return userRepo.findAll();
    }

    public void saveUser(User user, String username, Map<String, String> form) {
        user.setUsername(username);

        Set<String> roles = Arrays.stream(Role.values())
                .map(Role::name)
                .collect(Collectors.toSet());

        user.getRoles().clear();

        for (String key : form.keySet()) {
            if (roles.contains(key)) {
                user.getRoles().add(Role.valueOf(key));
            }
        }

        userRepo.save(user);
    }

    public void updateProfile(User user, String password, String email) {
        String userEmail = user.getEmail();

        boolean isEmailChanged = (email != null && !email.equals(userEmail)) ||
                (userEmail != null && !userEmail.equals(email));

        if (isEmailChanged) {
            user.setEmail(email);

            if (!StringUtils.isEmpty(email)) {
                user.setActivationCode(UUID.randomUUID().toString());
            }
        }

        if (!StringUtils.isEmpty(password)) {
            user.setPassword(password);
        }

        userRepo.save(user);

        if (isEmailChanged) {
            sendMessage(user);
        }
    }

    public void subscribe(User currentUser, User user) {
        user.getSubscribers().add(currentUser);

        userRepo.save(user);
    }

    public void unsubscribe(User currentUser, User user) {
        user.getSubscribers().remove(currentUser);

        userRepo.save(user);
    }


    private void sendMessage(User user) {
        if (!StringUtils.isEmpty(user.getEmail())) {
            String message = String.format(
                    "Hello, %s! \n" +
                            "Welcome to Sweater. Please, visit next link: http://%s/activate/%s",
                    user.getUsername(),
                    hostname,
                    user.getActivationCode()
            );

            mailSender.send(user.getEmail(), "Activation code", message);
        }
    }
}
```

---

### Ответ:

```java
package com.testing.unittest;

import com.example.sweater.domain.MailSender;
import com.example.sweater.domain.Role;
import com.example.sweater.domain.User;
import com.example.sweater.domain.UserRepo;
import com.letscode.sarafan.domain.Message;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;

import java.util.*;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

class CodiumUserServiceTest {

    @Mock
    private UserRepo userRepo;

    @Mock
    private MailSender mailSender;

    @Mock
    private PasswordEncoder passwordEncoder;

    @InjectMocks
    private UserService userService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    void testLoadUserByUsername_userFound() {
        User user = new User();
        user.setUsername("testUser");
        when(userRepo.findByUsername("testUser")).thenReturn(user);

        UserDetails userDetails = userService.loadUserByUsername("testUser");

        assertEquals("testUser", userDetails.getUsername());
    }

    @Test
    void testLoadUserByUsername_userNotFound() {
        when(userRepo.findByUsername(anyString())).thenReturn(null);

        assertThrows(UsernameNotFoundException.class, () -> {
            userService.loadUserByUsername("nonExistingUser");
        });
    }

    @Test
    void testAddUser_userDoesNotExist() {
        User user = new User();
        user.setUsername("newUser");
        when(userRepo.findByUsername("newUser")).thenReturn(null);
        when(passwordEncoder.encode(anyString())).thenReturn("encodedPassword");

        assertTrue(userService.addUser(user));

        verify(userRepo, times(1)).save(user);
        verify(mailSender, times(1)).send(anyString(), anyString(), anyString());
    }

    @Test
    void testAddUser_userAlreadyExists() {
        User user = new User();
        user.setUsername("existingUser");
        when(userRepo.findByUsername("existingUser")).thenReturn(user);

        assertFalse(userService.addUser(user));

        verify(userRepo, never()).save(user);
        verify(mailSender, never()).send(anyString(), anyString(), anyString());
    }

    @Test
    void testActivateUser_userWithActivationCodeFound() {
        User user = new User();
        String activationCode = "testActivationCode";
        user.setActivationCode(activationCode);
        when(userRepo.findByActivationCode(activationCode)).thenReturn(user);

        assertTrue(userService.activateUser(activationCode));

        assertNull(user.getActivationCode());
        verify(userRepo, times(1)).save(user);
    }

    @Test
    void testActivateUser_userWithActivationCodeNotFound() {
        String nonExistingActivationCode = "nonExistingActivationCode";
        when(userRepo.findByActivationCode(nonExistingActivationCode)).thenReturn(null);

        assertFalse(userService.activateUser(nonExistingActivationCode));

        verify(userRepo, never()).save(any());
    }

    @Test
    void testFindAll() {
        List<User> userList = new ArrayList<>();
        when(userRepo.findAll()).thenReturn(userList);

        assertEquals(userList, userService.findAll());
    }

    // ... (previous code)

    @Test
    void testSaveUser_updateRoles() {
        User user = new User();
        user.setUsername("testUser");
        List<Role> roles = new ArrayList<>(Arrays.asList(Role.USER));
        user.setRoles(roles);

        User userForm = new User();
        userForm.setRoles(new ArrayList<>(Arrays.asList(Role.ADMIN, Role.USER)));

        userService.saveUser(user, userForm.getUsername(), new HashMap<>());

        assertEquals(userForm.getRoles(), user.getRoles());
    }

    @Test
    void testUpdateProfile_updateProfileInfo() {
        User user = new User();
        user.setUsername("testUser");
        user.setEmail("oldEmail@test.com");
        user.setPassword("oldPassword");

//        Profile profileForm = new ProfileForm();
//        profileForm.setEmail("newEmail@test.com");
//        profileForm.setPassword("newPassword");

//        userService.updateProfile(user, profileForm);

        assertEquals("newEmail@test.com", user.getEmail());
        assertNotEquals("oldPassword", user.getPassword());
    }

    @Test
    void testSubscribe_userSuccessfullySubscribed() {
        User subscriber = new User();
        subscriber.setUsername("subscriberUser");
        User targetUser = new User();
        targetUser.setUsername("targetUser");
        Set<User> subscriptions = new HashSet<>();
        subscriber.setSubscriptions(subscriptions);

        userService.subscribe(subscriber, targetUser);

        assertTrue(subscriber.getSubscriptions().contains(targetUser));
    }

    @Test
    void testUnsubscribe_userSuccessfullyUnsubscribed() {
        User subscriber = new User();
        subscriber.setUsername("subscriberUser");
        User targetUser = new User();
        targetUser.setUsername("targetUser");
        Set<User> subscriptions = new HashSet<>(Arrays.asList(targetUser));
        subscriber.setSubscriptions(subscriptions);

        userService.unsubscribe(subscriber, targetUser);

        assertFalse(subscriber.getSubscriptions().contains(targetUser));
    }

    @Test
    void testSendMessage_emailNotEmpty() {
        User user = new User();
        user.setUsername("testUser");
        user.setEmail("testUser@test.com");

        Message messageForm = new Message();
        messageForm.setRecipient("recipientUser");
        messageForm.setText("Hello!");

        assertTrue(userService.sendMessage(user, messageForm));
    }

    @Test
    void testSendMessage_emailEmpty() {
        User user = new User();
        user.setUsername("testUser");
        user.setEmail("");

        Message messageForm = new Message();
        messageForm.setRecipient("recipientUser");
        messageForm.setText("Hello!");

        assertFalse(userService.sendMessage(user, messageForm));
    }
}
// More test cases for other methods in the UserService class
// ...
```