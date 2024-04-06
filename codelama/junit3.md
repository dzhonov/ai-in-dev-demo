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

To implement unit tests for the UserService class, you can use JUnit. Here is an example of how you can do this:
```java

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.Mockito.when;

@SpringBootTest
public class UserServiceTest {
    @Autowired
    private UserService userService;

    @MockBean
    private UserRepo userRepo;

    @MockBean
    private MailSender mailSender;

    @MockBean
    private PasswordEncoder passwordEncoder;

    @Test
    public void testLoadUserByUsername() {
        User user = new User();
        user.setUsername("test");
        user.setPassword("test");
        user.setEmail("test@example.com");

        when(userRepo.findByUsername("test")).thenReturn(user);

        UserDetails userDetails = userService.loadUserByUsername("test");

        assertEquals("test", userDetails.getUsername());
        assertEquals("test", userDetails.getPassword());
        assertEquals("test@example.com", userDetails.getEmail());
    }

    @Test
    public void testLoadUserByUsername_NotFound() {
        when(userRepo.findByUsername("test")).thenReturn(null);

        assertThrows(UsernameNotFoundException.class, () -> userService.loadUserByUsername("test"));
    }

    @Test
    public void testAddUser() {
        User user = new User();
        user.setUsername("test");
        user.setPassword("test");
        user.setEmail("test@example.com");

        when(userRepo.findByUsername("test")).thenReturn(null);
        when(passwordEncoder.encode("test")).thenReturn("test");

        boolean result = userService.addUser(user);

        assertEquals(true, result);
    }

    @Test
    public void testAddUser_AlreadyExists() {
        User user = new User();
        user.setUsername("test");
        user.setPassword("test");
        user.setEmail("test@example.com");

        when(userRepo.findByUsername("test")).thenReturn(user);

        boolean result = userService.addUser(user);

        assertEquals(false, result);
    }

    @Test
    public void testActivateUser() {
        User user = new User();
        user.setUsername("test");
        user.setPassword("test");
        user.setEmail("test@example.com");

        when(userRepo.findByActivationCode("test")).thenReturn(user);

        boolean result = userService.activateUser("test");

        assertEquals(true, result);
    }

    @Test
    public void testActivateUser_NotFound() {
        when(userRepo.findByActivationCode("test")).thenReturn(null);

        boolean result = userService.activateUser("test");

        assertEquals(false, result);
    }

    @Test
    public void testFindAll() {
        User user1 = new User();
        user1.setUsername("test1");
        user1.setPassword("test1");
        user1.setEmail("test1@example.com");

        User user2 = new User();
        user2.setUsername("test2");
        user2.setPassword("test2");
        user2.setEmail("test2@example.com");

        when(userRepo.findAll()).thenReturn(Arrays.asList(user1, user2));

        List<User> result = userService.findAll();
```
---


### Запрос:

complete the testFindAll test completely.

---

### Ответ:

Sure, here's the completed testFindAll test:

```java
public class UserServiceTest {
@Test
public void testFindAll() {
UserService userService = new UserService();
List<User> users = userService.findAll();
assertEquals(2, users.size());
assertEquals("John", users.get(0).getName());
assertEquals("Doe", users.get(0).getLastName());
assertEquals("john.doe@example.com", users.get(0).getEmail());
assertEquals("Jane", users.get(1).getName());
assertEquals("Doe", users.get(1).getLastName());
assertEquals("jane.doe@example.com", users.get(1).getEmail());
}
}
```

This test creates a UserService instance and calls the findAll method to retrieve a list of users. It then asserts that the list contains two users, and that the first user has the expected name, last name, and email address.