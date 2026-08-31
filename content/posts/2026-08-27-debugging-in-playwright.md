---
title: "Debugging in Playwright"
date: 2026-08-29
description: "Learn how to debug Playwright tests with Java and JUnit using headed mode, Playwright Inspector, API logs and Trace Viewer."
categories: ["QA"]
tags: ["QA", "Testing", "Playwright", "Java", "Maven", "JUnit", "Automation"]
image: "/images/playwright-debugging.png"
readTime: "8 min read"
---

## Introduction

End-to-end tests exercise several parts of a system at once: the browser, application code, network requests, test data and external services. A test can therefore fail even when its final assertion looks correct. The page may not have finished loading, a locator may match the wrong element, test data may have changed, an API request may have failed or a dialog may be blocking the next action.

Playwright reduces much of this uncertainty through auto-waiting and web-first assertions, but failures still need investigation. Its debugging capabilities allow us to:

* watch a test run in a visible browser;
* slow down browser actions;
* pause execution and inspect the page interactively;
* generate and test locators with Playwright Inspector;
* print detailed Playwright API logs;
* use browser developer tools; and
* record a trace that can be examined after a failure.

This article demonstrates these techniques with Playwright for Java, JUnit 5 and the Bookstore application.

## Prerequisites

We will use the same application and Playwright test setup created in [Capturing Screenshots in Playwright](../2026-08-25-capturing-screenshots-in-playwright-java-test-with-junit/). Before continuing, make sure that:

* the [Bookstore GitHub repository](https://github.com/nehajain216/bookstore) is cloned locally;
* the Bookstore application is running at `http://localhost:8080`;
* a test user exists in the application;
* Playwright, JUnit 5 and the Playwright browsers are installed; and
* the `BasePlaywrightTest` class and its browser setup are available in the test project.

The examples below use a login test similar to this one:

```java
class BookstoreLoginTest extends BasePlaywrightTest {

    @Test
    void shouldShowAnErrorForInvalidCredentials() {
        page.navigate("http://localhost:8080/login");

        page.getByLabel("Email").fill("Neha@gmail.com");
        page.getByLabel("Password").fill("incorrect-password");
        page.getByRole(
                AriaRole.BUTTON,
                new Page.GetByRoleOptions().setName("Sign in").setExact(true)
        ).click();

        assertThat(page.getByRole(AriaRole.ALERT))
                .hasText("Invalid email or password");
    }
}
```

## Run the test in headed mode

As covered in the previous articles, headed mode lets you watch the browser while a test runs. If an interaction happens too quickly to follow, add `setSlowMo()` temporarily:

```java
browser = playwright.chromium().launch(
        new BrowserType.LaunchOptions()
                .setHeadless(false)
                .setSlowMo(500)
);
```

The `500` value delays each Playwright operation by 500 milliseconds. Remove slow motion after debugging because it increases test execution time.

While running Chromium in headed mode, you can also open browser developer tools manually. Use the **Console** panel to investigate JavaScript errors and the **Network** panel to inspect failed requests and responses.

## Debug with Playwright Inspector

Playwright Inspector is an interactive tool for stepping through Playwright operations, viewing actionability logs and exploring locators. Start the Bookstore test in debug mode by setting the `PWDEBUG` environment variable:

```shell
PWDEBUG=1 ./mvnw -Dtest=BookstoreLoginTest test
```

To debug a single test method, specify the class and method names with `#`:

```shell
PWDEBUG=1 ./mvnw "-Dtest=LoginPageTest#shouldLoginOnValidCredentials" test
```

On Windows PowerShell, use:

```powershell
$env:PWDEBUG=1
./mvnw -Dtest=BookstoreLoginTest test
```

Debug mode opens the browser and Playwright Inspector. Use **Step over** to execute one Playwright operation at a time, **Resume** to continue the test and the locator picker to select an element on the page. The actionability log explains what Playwright waited for before an action, such as whether an element was visible, stable and enabled.

### Pause at a specific point

Add `page.pause()` where the application reaches the state that you want to investigate:

```java
page.navigate("http://localhost:8080/login");
page.getByLabel("Email").fill("Neha@gmail.com");

page.pause();

page.getByLabel("Password").fill("incorrect-password");
```

Run this test in headed mode. When execution reaches `page.pause()`, Inspector opens and the test waits until you resume it. You can inspect the DOM, try locators and confirm whether the page state matches the test's assumptions. Remove the pause after the problem is resolved so that automated runs do not stop indefinitely.

For a quick locator check, use `System.out.println()` to print its match count or state:

```java
Locator signInButton = page.getByRole(AriaRole.BUTTON,
        new Page.GetByRoleOptions().setName("Sign in").setExact(true));
System.out.println("Matches: " + signInButton.count());
System.out.println("Visible: " + signInButton.isVisible());
```

Remove temporary print statements after debugging.

## Enable Playwright API logs

When a failure is related to waiting or actionability, enable Playwright's API debug logs:

```shell
DEBUG=pw:api ./mvnw -Dtest=BookstoreLoginTest test
```

To enable API logs for a single test method, specify the class and method names:

```shell
DEBUG="pw:api" ./mvnw "-Dtest=LoginPageTest#shouldLoginAsAdminAndOpenDashboard" test
```

The output shows operations such as navigation, locator resolution, clicks and auto-waiting. For example, the log can show that Playwright found the **Sign in** button but continued waiting because it was not yet visible or enabled.

PowerShell users can set the variable before running Maven:

```powershell
$env:DEBUG="pw:api"
./mvnw -Dtest=BookstoreLoginTest test
```

API logs are verbose, so enable them for a failing test or class rather than for an entire test suite unless broader output is necessary.

## Record and inspect a Playwright trace

Headed mode and Inspector are best for a failure that can be reproduced locally. A trace is more useful for intermittent failures and CI runs because it records the test for later inspection. A Playwright trace can include DOM snapshots, screenshots and source information for each action.

The following test starts tracing before submitting invalid login credentials. Its intentionally incorrect assertion produces a failure to investigate:

```java
@Test
void shouldTraceWhenLoginAssertionFails() {
    context.tracing().start(new Tracing.StartOptions()
            .setScreenshots(true)
            .setSnapshots(true)
            .setSources(true));
    try {
        submitLogin("neha@gmail.com", "neha1");
        Locator errorMessage = page.getByRole(AriaRole.ALERT);
        assertThat(errorMessage).hasText("Wrong error message.");
    } finally {
        context.tracing().stop(new Tracing.StopOptions()
                .setPath(Paths.get("login-assertion-failure-trace.zip")));
    }
}
```

The `finally` block stops tracing and saves `login-assertion-failure-trace.zip` even when the assertion fails. Open the trace with the Playwright CLI that belongs to the Java project:

```shell
./mvnw exec:java \
  -Dexec.mainClass=com.microsoft.playwright.CLI \
  -Dexec.args="show-trace login-assertion-failure-trace.zip"
```

If you are using Java with Maven, you can also use the Node-based Playwright command. Although the tests are written in Java, Trace Viewer is provided through the Playwright tooling:

```shell
npx playwright show-trace login-assertion-failure-trace.zip
```

This option requires Node.js and npm to be installed.

Trace Viewer provides a timeline of Playwright actions. Select an action to inspect its before-and-after DOM snapshots, locator, logs, source location, network activity and console output. Because traces can contain page content and application data, handle CI trace artifacts according to the project's security and retention policies.

## A practical debugging workflow

When the Bookstore test fails, investigate it in this order:

1. Read the assertion error and Playwright call log. They often identify the failing locator or unmet condition immediately.
2. Run only the failing test in headed mode.
3. Add slow motion or `page.pause()` if the failure happens too quickly to observe.
4. Use Inspector to check the locator and the element's actionability state.
5. Enable `DEBUG=pw:api` when waiting behavior remains unclear.
6. Check browser console and network activity for application-side failures.
7. Record a trace when the issue is intermittent, occurs only in CI or needs to be shared with another developer.

Avoid fixing timing problems by adding `Thread.sleep()`. A fixed delay makes tests slower and still may not be long enough on a busy machine. Use Playwright locators and web-first assertions so Playwright waits for the required state:

```java
assertThat(page.getByRole(AriaRole.ALERT))
        .hasText("Invalid email or password");
```

## Conclusion

Playwright provides several complementary ways to debug end-to-end tests. Headed mode and slow motion make the browser behavior visible, Inspector and `page.pause()` help examine actions interactively, API logs explain auto-waiting, developer tools expose browser-side failures and Trace Viewer preserves evidence from CI or intermittent runs.

Start with the failure message and the smallest failing test, then add the debugging tool that answers the next question. This approach keeps Playwright tests reliable without hiding synchronization problems behind fixed delays.
