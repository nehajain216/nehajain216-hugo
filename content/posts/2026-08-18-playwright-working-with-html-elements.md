---
title: "Playwright working with HTML elements"
date: 2026-08-19
description: "Learn how to locate form elements, performing actions and validating using Playwright."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/playwrigt-html-elements.png"
readTime: "6 min read"
---

Playwright locators provide a reliable way to find and interact with HTML elements. In this article, we will use the [Bookstore application's](https://github.com/nehajain216/bookstore) login page as a practical example to locate links, text fields, buttons and visible text. We will then perform actions such as `click()` and `fill()` and validate element states using Playwright assertions.

## Prerequisites

Before starting, make sure that:

* The [Bookstore GitHub repository](https://github.com/nehajain216/bookstore) is cloned locally.
* The Bookstore application is running at `http://localhost:8080`.
* A test user exists in the application.
* Playwright, JUnit and the Playwright browsers are installed.
* The `BasePlaywrightTest` from the [previous post](https://nehajain.netlify.app/posts/2026-08-05-getting-started-with-playwright-java-maven-junit/) is available in the test project.

For this test, we will sign in as a regular user with the email `neha@gmail.com` and password `siva`. After a successful login, the username `Neha` should appear on the page.

## The login test scenario

Our test will perform the following steps:

1. Navigate to `http://localhost:8080/books`.
2. Click the **Login** link on the home page.
3. Verify that the browser navigates to `http://localhost:8080/login`.
4. Enter an email address and password.
5. Click the **Sign in** button.
6. Verify that the user is redirected to `http://localhost:8080/books`.
7. Verify that the username is visible in the top-right area of the page.

## Locating HTML elements

A Playwright `Locator` represents a way to find one or more elements on a page. It does not store a fixed copy of an element. Playwright resolves the locator when an action or assertion is performed, so it can find the element even when the page changes after the locator is created.

Playwright provides several locator methods. Whenever possible, prefer locators based on the way a user identifies an element: its role, label or visible text.

### Common locator methods and when to use them

The following locator methods cover most interactions in a web application.

#### 1. Locate by role

Use `getByRole()` for interactive and semantic elements such as links, buttons, headings, checkboxes and radio buttons. It uses an element's ARIA role and accessible name, so the locator closely matches how users and assistive technologies understand the page.

```java
Locator loginLink = page.getByRole(
        AriaRole.LINK,
        new Page.GetByRoleOptions().setName("Login").setExact(true)
);

Locator signInButton = page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Sign in").setExact(true)
);
```

Prefer this method when the element has a clear role and accessible name.

#### 2. Locate by label

Use `getByLabel()` for form controls associated with a `<label>`, such as text fields, password fields, checkboxes and select elements.

```java
Locator emailField = page.getByLabel("Email");
Locator passwordField = page.getByLabel("Password");
```

This is usually the clearest way to locate a form control because it reflects the label visible to the user.

#### 3. Locate by visible text

Use `getByText()` for non-interactive content such as messages, usernames, table cells and other text displayed on the page.

```java
Locator username = page.getByText(
        "Neha",
        new Page.GetByTextOptions().setExact(true)
);
```

Use an exact match when similar or longer text could also appear on the page. For buttons and links, prefer `getByRole()` instead because it describes both the element type and its name.

#### 4. Locate by ID or another CSS selector

If an element has a unique and stable ID, it can be located with a short CSS selector. Prefix the ID with `#` when passing it to `locator()`:

```java
Locator emailById = page.locator("#email");
```

This locator targets an element such as:

```html
<input id="email" name="email" type="email">
```

An ID locator is simple and useful when the ID is unique, predictable and intentionally kept stable. Avoid IDs generated dynamically by a framework because they may change between page loads or application releases.

The `locator()` method also accepts other CSS selectors. For example, an element can be selected by its `name` attribute or class:

```java
Locator passwordByName = page.locator("[name='password']");
Locator loginForm = page.locator(".login-form");
```

Stable IDs and attributes are generally better choices than classes because classes often change during visual redesigns. Make sure the selector uniquely identifies the intended element.

Playwright also supports XPath through `locator()`:

```java
Locator emailByXPath = page.locator("xpath=//input[@id='email']");
```

XPath can help with legacy pages or complex document structures, but it should usually be a fallback. XPath expressions are harder to read and can become fragile when the HTML structure changes. In this example, `getByLabel("Email")` or `locator("#email")` is simpler and more maintainable.

### Choose a locator based on the test's intent

The choice of locator depends on whether a test should follow the page's implementation structure or its user-observable behavior.

An implementation-focused locator identifies an element through technical details such as its ID, attributes or position in the HTML. A user-focused locator identifies it through information available to the user, such as its role, label or visible text.

Keep the following trade-offs in mind:

* An ID locator may break after an internal refactor even when the interface continues to work correctly.
* A role locator can reveal accessibility problems that an ID locator would miss. For example, `locator("#sign-in")` can match a clickable `<div id="sign-in">`, even though the element does not have button semantics.
* Role and text locators may break when visible wording changes or the application is translated. A stable ID can be more suitable in those situations.
* IDs work well for implementation-level UI tests when they are unique and intentionally maintained as part of the application's testing contract.
* User-observable locators work well for end-to-end tests because they interact with the interface in a way that is closer to a real user.

A practical strategy is to prefer roles, labels and visible text for important user journeys. Use a stable ID or test ID when a semantic locator is unavailable, ambiguous or expected to change because of localization.

Choose a locator based on what the test is intended to protect. Use IDs when the test needs to depend on the page's technical structure. Use roles, labels and visible text when the test should verify that users can identify and interact with the interface.

### Link: locate, verify and click

An HTML link has the ARIA role `link`. We can locate the **Login** link by its accessible name, verify that it is visible and then click it:

```java
Locator loginLink = page.getByRole(
        AriaRole.LINK,
        new Page.GetByRoleOptions().setName("Login").setExact(true)
);

assertThat(loginLink).isVisible();
loginLink.click();
```

Using a role-based locator makes the test readable and encourages accessible HTML. Before clicking, Playwright automatically waits for the link to become visible, enabled and stable.

### Text fields: locate, fill and validate values

If the login form uses labels such as `<label for="email">Email</label>`, locate the fields with `getByLabel()`:

```java
Locator emailField = page.getByLabel("Email");
Locator passwordField = page.getByLabel("Password");

assertThat(emailField).isVisible();
assertThat(passwordField).isVisible();

emailField.fill("neha@gmail.com");
passwordField.fill("siva");

assertThat(emailField).hasValue("neha@gmail.com");
assertThat(passwordField).hasValue("siva");
```

The `fill()` action clears any existing content before entering the supplied text. The `hasValue()` assertions confirm that the expected values were entered. Locating fields by label is usually more stable than selecting them with CSS classes, and it encourages meaningful accessible labels.

If the page does not have associated labels, add them to the HTML when possible. As a fallback, fields can be located by placeholder or test ID:

```java
page.getByPlaceholder("you@example.com");
page.getByTestId("email");
```

### Button: locate, verify and click

A submit button can be located by its button role and accessible name. Before clicking it, verify that the button is enabled:

```java
Locator signInButton = page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Sign in").setExact(true)
);

assertThat(signInButton).isEnabled();
signInButton.click();
```

The `setExact(true)` option prevents a similar label, such as **Sign in with Google**, from matching accidentally.

### Text content: locate and verify visibility

After the form is submitted, locate the displayed username by its visible text and verify that it appears on the page:

```java
Locator username = page.getByText(
        "Neha",
        new Page.GetByTextOptions().setExact(true)
);

assertThat(username).isVisible();
```

`getByText()` is useful for locating non-interactive content such as headings, messages and usernames. The exact match prevents Playwright from matching a longer piece of text that happens to contain `Neha`.

## Write the complete login test

Create a test file at:

```text
src/test/java/dev/nehajain/bookstore/LoginPageTest.java
```

Add the following code:

```java
package dev.nehajain.bookstore;

import com.microsoft.playwright.Locator;
import com.microsoft.playwright.Page;
import com.microsoft.playwright.options.AriaRole;
import org.junit.jupiter.api.Test;

import static com.microsoft.playwright.assertions.PlaywrightAssertions.assertThat;

class LoginPageTest extends BasePlaywrightTest {

    private static final String BASE_URL = "http://localhost:8080";

    @Test
    void shouldLoginWithValidCredentials() {
        page.navigate(BASE_URL + "/books");

        Locator loginLink = page.getByRole(
                AriaRole.LINK,
                new Page.GetByRoleOptions().setName("Login").setExact(true)
        );
        assertThat(loginLink).isVisible();
        loginLink.click();

        assertThat(page).hasURL(BASE_URL + "/login");

        Locator emailField = page.getByLabel("Email");
        Locator passwordField = page.getByLabel("Password");
        Locator signInButton = page.getByRole(
                AriaRole.BUTTON,
                new Page.GetByRoleOptions().setName("Sign in").setExact(true)
        );

        assertThat(emailField).isVisible();
        assertThat(passwordField).isVisible();
        assertThat(signInButton).isEnabled();

        emailField.fill("neha@gmail.com");
        passwordField.fill("siva");

        assertThat(emailField).hasValue("neha@gmail.com");
        assertThat(passwordField).hasValue("siva");

        signInButton.click();

        assertThat(page).hasURL(BASE_URL + "/books");
        assertThat(page.getByText("Neha", new Page.GetByTextOptions().setExact(true)))
                .isVisible();
    }
}
```

## Understanding the actions

The test uses three basic browser actions:

* `navigate()` opens a URL in the browser.
* `fill()` clears an input and enters the supplied value.
* `click()` clicks an element after Playwright confirms that it is visible, enabled and stable.

These actions use Playwright's built-in auto-waiting. In most tests, there is no need to add fixed delays such as `Thread.sleep()`.

## Understanding the validations

The test checks both navigation and visible page state:

```java
assertThat(page).hasURL(BASE_URL + "/login");
assertThat(page).hasURL(BASE_URL + "/books");
```

`hasURL()` waits until the current URL matches the expected value. This is useful after clicking a link or submitting a form because navigation may take a moment.

The final assertion confirms that the authenticated user's name is rendered on the page:

```java
assertThat(page.getByText("Neha", new Page.GetByTextOptions().setExact(true)))
        .isVisible();
```

Checking the URL alone is not enough. The visible username provides additional evidence that the application created an authenticated session and displayed the signed-in state.

If the username has a unique semantic element, prefer an even more specific locator. For example, when it is part of a user menu button:

```java
assertThat(page.getByRole(
        AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Neha").setExact(true)
)).isVisible();
```

## Run the test

Start the application, then run the test from the IDE or with Maven:

```bash
mvn test -Dtest=LoginPageTest
```

The browser should open the books page, navigate to the login form, submit the credentials and return to the books dashboard. The test passes when the final URL and username are both correct.

## Keep credentials outside the test

Hard-coded credentials make an example easy to follow, but real test credentials should come from environment variables or another secure configuration source:

```java
String email = System.getenv("TEST_USER_EMAIL");
String password = System.getenv("TEST_USER_PASSWORD");

emailField.fill(email);
passwordField.fill(password);
```

Do not commit real passwords to source control.

## Conclusion

The Bookstore login page provides a practical example of locating and interacting with common HTML elements. Role and label locators keep tests close to how users interact with a page, while Playwright's auto-waiting assertions make them reliable without manual delays.

You can apply the same techniques to other parts of a web application. Locate elements using their accessible roles or labels, perform the required actions and assert that each element reaches the expected state.
