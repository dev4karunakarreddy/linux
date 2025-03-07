# Google Java Style Guide

The **Google Java Style Guide** provides a set of coding conventions that help maintain consistency, readability, and best practices in Java development.

## 1. File Structure

### 1.1 File Encoding
- All Java source files should be encoded in **UTF-8**.
- This ensures compatibility across different operating systems and environments.
- Using UTF-8 prevents issues with special characters and ensures consistency.

### 1.2 File Name
- The file name should match the **class name** exactly, including case sensitivity.
- It should follow **PascalCase** (each word starts with an uppercase letter, no underscores or dashes).
```java
public class MyJavaClass {
    // Code here
}
```
- This helps in maintaining clarity and avoids confusion in large projects.

## 2. Source Code Formatting

### 2.1 Indentation
- **Use spaces, not tabs**.
- Every indentation level should be exactly **4 spaces**.
```java
public void myMethod() {
    int x = 5;  // Indented 4 spaces
    if (x > 0) {
        System.out.println("Positive");  // Further indented 4 spaces
    }
}
```

### 2.2 Line Length
- **Maximum 100 characters per line**.
- If a line exceeds 100 characters, break it logically.
- Helps in maintaining readability across different screen sizes.

### 2.3 Wrapping Lines
- Break the line **before** an operator (`+`, `&&`, `||`, etc.) or after a comma.
- Helps maintain readability and avoids horizontal scrolling.
```java
String message = "This is a very long message that should be wrapped " +
                 "properly to maintain readability.";
```

## 3. Naming Conventions

### 3.1 Class and Interface Names
- Always use **PascalCase**.
- Start each word with an uppercase letter.
```java
public class AccountManager {}
```
- Helps in distinguishing classes from variables and methods.

### 3.2 Method Names
- Use **camelCase**, starting with a lowercase letter.
- Name should clearly describe the function’s purpose.
```java
public void processTransaction() {}
```

### 3.3 Constant Variables
- Use **UPPER_SNAKE_CASE** and `static final`.
- Constants should be descriptive and not generic like `NUM1`.
```java
private static final int MAX_USERS = 100;
```
- Helps in making constants easily identifiable.

### 3.4 Local Variables and Parameters
- Use **camelCase**.
- Variable names should be descriptive.
```java
int maxSpeed = 120;
```
- This makes the code self-explanatory.

## 4. Comments

### 4.1 Javadoc Comments
- Required for all **public** classes and methods.
- Use `@param` and `@return` to describe parameters and return values.
```java
/**
 * Calculates interest for a given amount.
 * @param principal the principal amount
 * @return interest value
 */
public double calculateInterest(double principal) {}
```

### 4.2 Inline Comments
- Use `//` for short explanations within the code.
```java
// Calculate the area of the circle
area = Math.PI * radius * radius;
```

## 5. Coding Best Practices

### 5.1 Use Braces for Control Statements
- Always use braces `{}` even for single-line blocks.
```java
if (condition) {
    executeAction();
}
```
- Prevents logical errors in future modifications.

### 5.2 Use `final` Where Applicable
- Helps in enforcing immutability.
- Useful for security and thread safety.
```java
public final class Constants {}
```

### 5.3 Avoid Magic Numbers
- Avoid using raw numbers in logic.
- Define constants instead.
```java
private static final int LEGAL_AGE = 18;
if (age > LEGAL_AGE) {}
```

## 6. Exception Handling

### 6.1 Use Specific Exceptions
- Do not use generic `Exception` or `Throwable`.
- Use specific exceptions like `IllegalArgumentException`, `IOException`.
```java
if (value == null) {
    throw new NullPointerException("Value cannot be null");
}
```

### 6.2 Avoid Catching Generic Exceptions
```java
// Bad
catch (Exception e) {}

// Good
catch (IOException e) {}
```
- Helps in debugging by handling specific error cases.

## 7. Multithreading and Concurrency (Advanced)

### 7.1 Use `synchronized` for Critical Sections
- Prevents race conditions.
```java
public synchronized void updateBalance() {
    balance += 100;
}
```

### 7.2 Prefer `ConcurrentHashMap` over `HashMap` in Multithreaded Code
```java
ConcurrentHashMap<Integer, String> map = new ConcurrentHashMap<>();
```
- Avoids concurrent modification exceptions.

### 7.3 Use `volatile` for Visibility
```java
private volatile boolean running = true;
```
- Ensures variable updates are visible across threads.

## 8. Java Streams and Functional Programming (Advanced)

### 8.1 Using Streams for Collection Operations
- Makes collection processing cleaner and more efficient.
```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.stream().filter(name -> name.startsWith("A")).forEach(System.out::println);
```

### 8.2 Using `Optional` to Avoid Null Checks
```java
Optional<String> name = Optional.ofNullable(getUserName());
name.ifPresent(System.out::println);
```
- Reduces `NullPointerException` risks.

## 9. Unit Testing and Code Quality

### 9.1 Use JUnit for Testing
- Ensures correctness of logic.
```java
@Test
public void testAddition() {
    assertEquals(4, MathUtils.add(2, 2));
}
```

### 9.2 Use Mocking Frameworks like Mockito
```java
Mockito.when(mockObject.getValue()).thenReturn("Mocked Value");
```
- Helps in isolating dependencies during testing.

## Conclusion
Following the **Google Java Style Guide** ensures **readability**, **maintainability**, and **efficiency** in Java projects. By adhering to these guidelines, developers of all levels can write cleaner and more consistent code, improving collaboration and reducing errors.

