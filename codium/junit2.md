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

```java
package com.testing.unittest;


import com.example.sweater.domain.Person;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import java.util.Date;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.when;

class CodiumValidatorTest {

    @Test
    void testIsValidPerson() {
        Person person = new Person();
        person.setFirstName("John");
        person.setLastName("Doe");
        person.setDateOfBirth(new Date());

        boolean result = Validator.isValidPerson(person);

        assertTrue(result);
    }

    @Test
    void testIsValidName() {
        assertTrue(Validator.isValidName("John"));
        assertFalse(Validator.isValidName(""));
        assertFalse(Validator.isValidName("123"));
    }

    @Test
    void testIsValidDateOfBirth() {
        Date currentDate = new Date();
        Date futureDate = new Date(currentDate.getTime() + 1000000000); // A future date
        assertFalse(Validator.isValidDateOfBirth(futureDate));
    }

    @Test
    void testIsValidPlaceOfBirth() {
        assertTrue(Validator.isValidPlaceOfBirth("New York"));
        assertFalse(Validator.isValidPlaceOfBirth(""));
    }

    @Test
    void testIsValidNationality() {
        assertTrue(Validator.isValidNationality("American"));
        assertFalse(Validator.isValidNationality(""));
    }

    @Test
    void testIsValidGender() {
        assertTrue(Validator.isValidGender("мужской"));
        assertTrue(Validator.isValidGender("женский"));
        assertFalse(Validator.isValidGender("other"));
    }

    @Test
    void testIsValidAddress() {
        assertTrue(Validator.isValidAddress("123 Street, City"));
        assertFalse(Validator.isValidAddress(""));
    }

    @Test
    void testIsValidPhoneNumber() {
        assertTrue(Validator.isValidPhoneNumber("+1234567890"));
        assertFalse(Validator.isValidPhoneNumber("invalid"));
    }

    @Test
    void testIsValidEmail() {
        assertTrue(Validator.isValidEmail("john.doe@example.com"));
        assertFalse(Validator.isValidEmail("invalid"));
    }

    @Test
    void testIsValidMaritalStatus() {
        assertTrue(Validator.isValidMaritalStatus("холост"));
        assertTrue(Validator.isValidMaritalStatus("замужем"));
        assertFalse(Validator.isValidMaritalStatus("other"));
    }

    @Test
    void testIsValidOccupation() {
        assertTrue(Validator.isValidOccupation("Engineer"));
        assertFalse(Validator.isValidOccupation(""));
    }

    @Test
    void testIsValidEducation() {
        assertTrue(Validator.isValidEducation("Bachelor's degree"));
        assertFalse(Validator.isValidEducation(""));
    }

    @Test
    void testIsValidDocumentType() {
        assertTrue(Validator.isValidDocumentType("Passport"));
        assertFalse(Validator.isValidDocumentType(""));
    }

    @Test
    void testIsValidDocumentNumber() {
        assertTrue(Validator.isValidDocumentNumber("AB123456"));
        assertFalse(Validator.isValidDocumentNumber(""));
    }

    @Test
    void testIsValidDocumentDates() {
        Date currentDate = new Date();
        Date futureDate = new Date(currentDate.getTime() + 1000000000); // A future date
        assertFalse(Validator.isValidDocumentDates(futureDate, currentDate));
    }
}
```