## JUNIT 2 Validator

### Запрос:

Cover the code below with unit tests. Provide 100% coverage, use junit.
```java

import java.util.Date;
import java.util.regex.Pattern;

public class Validator {
    public static boolean isValidPerson(Person person) {
        return isValidName(person.getFirstName()) &&
                isValidName(person.getLastName()) &&
                isValidDateOfBirth(person.getDateOfBirth()) &&
                isValidPlaceOfBirth(person.getPlaceOfBirth()) &&
                isValidNationality(person.getNationality()) &&
                isValidGender(person.getGender()) &&
                isValidAddress(person.getAddress()) &&
                isValidPhoneNumber(person.getPhoneNumber()) &&
                isValidEmail(person.getEmail()) &&
                isValidMaritalStatus(person.getMaritalStatus()) &&
                isValidOccupation(person.getOccupation()) &&
                isValidEducation(person.getEducation()) &&
                isValidDocumentType(person.getDocumentType()) &&
                isValidDocumentNumber(person.getDocumentNumber()) &&
                isValidDocumentDates(person.getDocumentIssueDate(), person.getDocumentExpirationDate());
    }

    public static boolean isValidName(String name) {
        return !name.isEmpty() && Pattern.matches("^[a-zA-Zа-яА-ЯёЁ\\\\s]+$", name);
    }

    public static boolean isValidDateOfBirth(Date dateOfBirth) {
        Date currentDate = new Date();
        return !dateOfBirth.after(currentDate);
    }
1
    public static boolean isValidPlaceOfBirth(String placeOfBirth) {
        return !placeOfBirth.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidNationality(String nationality) {
        return !nationality.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidGender(String gender) {
        return gender != null && (gender.equalsIgnoreCase("мужской") || gender.equalsIgnoreCase("женский"));
    }

    public static boolean isValidAddress(String address) {
        return !address.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidPhoneNumber(String phoneNumber) {
        return !phoneNumber.isEmpty() && Pattern.matches("^\\\\+?\\\\d{1,4}[-.\\\\s]?\\\\(?(\\\\d{1,3})\\\\)?[-.\\\\s]?\\\\d{1,4}[-.\\\\s]?\\\\d{1,9}$", phoneNumber);
    }

    public static boolean isValidEmail(String email) {
        return !email.isEmpty() && Pattern.matches("^[a-zA-Z0-9+_.-]+@[a-zA-Z0-9.-]+$", email);
    }

    public static boolean isValidMaritalStatus(String maritalStatus) {
        return maritalStatus != null && (maritalStatus.equalsIgnoreCase("холост") ||
                maritalStatus.equalsIgnoreCase("замужем") ||
                maritalStatus.equalsIgnoreCase("женат") ||
                maritalStatus.equalsIgnoreCase("разведен") ||
                maritalStatus.equalsIgnoreCase("вдовец") ||
                maritalStatus.equalsIgnoreCase("вдова"));
    }

    public static boolean isValidOccupation(String occupation) {
        return !occupation.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidEducation(String education) {
        return !education.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidDocumentType(String documentType) {
        return !documentType.isEmpty(); // Можно добавить дополнительные проверки
    }

    public static boolean isValidDocumentNumber(String documentNumber) {
        return !documentNumber.isEmpty() && Pattern.matches("^[a-zA-Z0-9]+$", documentNumber);
    }

    public static boolean isValidDocumentDates(Date issueDate, Date expirationDate) {
        return !issueDate.after(expirationDate);
    }
}
```

---

### Ответ:

To cover the code with unit tests, you can use JUnit to write test cases for each method. Here is an example of how you can write unit tests for the Validator class:

```java
import org.junit.Test;
import static org.junit.Assert.*;

public class ValidatorTest {
   @Test
   public void testIsValidPerson() {
       Person person = new Person();
       person.setFirstName("John");
       person.setLastName("Doe");
       person.setDateOfBirth(new Date());
       person.setPlaceOfBirth("New York");
       person.setNationality("American");
       person.setGender("Male");
       person.setAddress("123 Main St");
       person.setPhoneNumber("123-456-7890");
       person.setEmail("johndoe@example.com");
       person.setMaritalStatus("Married");
       person.setOccupation("Software Engineer");
       person.setEducation("Bachelor's Degree");
       person.setDocumentType("Passport");
       person.setDocumentNumber("123456789");
       person.setDocumentIssueDate(new Date());
       person.setDocumentExpirationDate(new Date());

       assertTrue(Validator.isValidPerson(person));
   }

   @Test
   public void testIsValidName() {
       assertTrue(Validator.isValidName("John Doe"));
       assertFalse(Validator.isValidName(""));
       assertFalse(Validator.isValidName("123456789"));
   }

   @Test
   public void testIsValidDateOfBirth() {
       Date currentDate = new Date();
       assertTrue(Validator.isValidDateOfBirth(currentDate));
       assertFalse(Validator.isValidDateOfBirth(new Date(currentDate.getTime() - 10000)));
   }

   @Test
   public void testIsValidPlaceOfBirth() {
       assertTrue(Validator.isValidPlaceOfBirth("New York"));
       assertFalse(Validator.isValidPlaceOfBirth(""));
   }

   @Test
   public void testIsValidNationality() {
       assertTrue(Validator.isValidNationality("American"));
       assertFalse(Validator.isValidNationality(""));
   }

   @Test
   public void testIsValidGender() {
       assertTrue(Validator.isValidGender("Male"));
       assertFalse(Validator.isValidGender(""));
   }

   @Test
   public void testIsValidAddress() {
       assertTrue(Validator.isValidAddress("123 Main St"));
       assertFalse(Validator.isValidAddress(""));
   }

   @Test
   public void testIsValidPhoneNumber() {
       assertTrue(Validator.isValidPhoneNumber("123-456-7890"));
       assertFalse(Validator.isValidPhoneNumber(""));
   }

   @Test
   public void testIsValidEmail() {
       assertTrue(Validator.isValidEmail("johndoe@example.com"));
       assertFalse(Validator.isValidEmail(""));
   }

   @Test
   public void testIsValidMaritalStatus() {
       assertTrue(Validator.isValidMaritalStatus("Married"));
       assertFalse(Validator.isValidMaritalStatus(""));
   }

   @Test
   public void testIsValidOccupation() {
       assertTrue(Validator.isValidOccupation("Software Engineer"));
       assertFalse(Validator.isValidOccupation(""));
   }

   @Test
   public void testIsValidEducation() {
       assertTrue(Validator.isValidEducation("Bachelor's Degree"));
       assertFalse(Validator.isValidEducation(""));
   }

   @Test
   public void testIsValidDocumentType() {
       assertTrue(Validator.isValidDocumentType("Passport"));
       assertFalse(Validator.isValidDocumentType(""));
   }

   @Test
   public void testIsValidDocumentNumber() {
       assertTrue(Validator.isValidDocumentNumber("123456789"));
       assertFalse(Validator.isValidDocumentNumber(""));
   }
```