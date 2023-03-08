---
title: "Getting Started With REST Assured for API Testing"
thumbnailImagePosition: left
thumbnailImage: /images/rest-assured.jpeg
coverImage: /images/rest-assured.jpeg
metaAlignment: center
coverMeta: out
date: 2023-03-08
draft: true
categories:
- qa
- testing
- rest assured
- api
- automation
---

In the world of software development, APIs (Application Programming Interfaces) play a crucial role in connecting different software components and allowing them to communicate with 
each other. As APIs have become increasingly popular, API testing has also become a crucial aspect of software testing.
In this blog post, we'll discuss the benefits of using Rest Assured for API testing, as well as how to get started with this powerful testing framework.

## What is Rest assured API testing?

Rest assured is an open-source Java-based library that is used for testing RESTful APIs. It provides a simple and intuitive API for creating HTTP requests and handling responses. Rest assured API testing allows to automate API testing and ensure that the API is functioning correctly. It can be used for both functional and non-functional testing.

## Advantages of using Rest assured API testing

* **Easy to use:** Rest Assured has a user-friendly API that makes it easy to write and maintain tests.

* **Support for different HTTP methods:** Rest Assured supports all the standard HTTP methods, including GET, POST, PUT, DELETE, and more.

* **Integrates with other tools:** Rest Assured integrates with popular testing frameworks like JUnit and TestNG, as well as with tools like Maven and Gradle.

* **Provides detailed reporting:** Rest assured provides detailed reporting of API testing results, making it easy to identify and fix issues quickly.

* **Built-in assertions:** Rest Assured provides a variety of built-in assertions to check the response of an API, including status codes, response body, headers, and more.

* **Supports automation:** Rest assured API testing can be automated, making it possible to run tests as part of a continuous integration and delivery (CI/CD) pipeline.

## Getting started with Rest Assured

To get started with Rest Assured, you'll need to have a few things in place:

### Prerequisites

* Java Development Kit (JDK) installed
* Eclipse or IntelliJ IDEA installed
* Maven installed

Once you have these prerequisites, you can start writing your first Rest Assured test. 

Here's a simple example:

### Step 1: Add Rest assured dependencies 

First, let's create a new Maven project in Eclipse or IntelliJ IDEA. Once the project is created, add the following dependency to the pom.xml file:

```shell
<dependency>
    <groupId>io.rest-assured</groupId>
    <artifactId>rest-assured</artifactId>
    <version>5.3.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.9.2</version>
    <scope>test</scope>
</dependency>
```
This will import Rest Assured and Junit into our project.

### Step 2: Write a simple API test

Next, you can write a simple Rest assured API test. Here's an example:

```java
public class ExampleAPITest {
    @Test
    void testStatusCode() {
        given()
            .when()
                .get("http://example.com/api/users")
            .then()
                .statusCode(200);
    }
}
```

In this example, we're making a GET request to the "http://example.com/api/users" endpoint and checking 
that the response has a status code of 200. The ```given()``` method sets up the request, and the ```then()``` method checks the 
response.

### Step 3: Run the API test

To run the test, right-click on the Test class and select Run As and check the results.

Let's explore the basic HTTP methods - **GET, PUT, POST, and DELETE**

We will use the following endpoints for our API testing:

* POST: /api/users
* GET: /api/users/{userId}
* PUT: /api/users/{userId}
* DELETE: /api/users/{userId}


### POST Method:
The POST method is used to create a new resource on the server.

To test the POST method, we need to send a request to create a new user. We will use the given URL and JSON payload.

**URL:** https://example.com/api/users

**JSON Payload:** 
```
{
    "name": "Smith", 
    "email": "smith@example.com", 
    "age": "23"
}
```
```java
@Test
void shouldCreateUser() {
        given()
            .contentType("application/json")
            .body("{\"name\": \"Smith\", \"email\": \"smith@example.com\", \"age\": \"23\"}")
        .when()
            .post("https://example.com/api/users")
        .then()
            .statusCode(201)
            .body("name" equalTo("Smith"))
            .body("email" equalTo("smith@example.com"))
            .body("age" equalTo("23"))
        ;
}
```
We are using the given(), when(), and then() methods provided by REST Assured. 

Using **given()** we are setting the content type and request body

Using **when()** we are sending the post() request with endpoint URL and 

Using **then()** we are verifying the response

### GET Method:
The GET method is used to retrieve information from the server.

To test the GET method, we need to retrieve user information using the given URL and user ID.

**URL:** https://example.com/api/users/{userId}

```java
@Test
void shouldGetUserById() {
        given()
            .contentType("application/json")
        .when()
            .get("https://example.com/api/users/{userId}",1)
        .then()
            .statusCode(201)
            .body("name" equalTo("Smith"))
            .body("email" equalTo("smith@example.com"))
            .body("age" equalTo("23"))
        ;
}
```
We are using the given(), when(), and then() methods provided by REST Assured.

Using **given()** we are setting the content type

Using **when()** we are sending the get() request with endpoint URL and path variable(s)

Using **then()** we are verifying the response

### PUT Method:
The PUT method is used to update an existing resource on the server.

To test the PUT method, we need to update user information using the given URL and user ID.

**URL:** https://example.com/api/users/{userId}

**JSON Payload:** 
```
{
    "name": "Smith Rich", 
    "email": "smith.rich@example.com", 
    "age": "26"
}
```

```java
@Test
void shouldUpdateUser() {
        given()
            .contentType("application/json")
            .body("{\"name\": \"Smith Rich\", \"email\": \"smith.rich@example.com\", \"age\": \"26\"}")
        .when()
            .put("https://example.com/api/users/{userId}",1)
        .then()
            .statusCode(200)
            .body("name" equalTo("Smith Rich"))
            .body("email" equalTo("smith.rich@example.com"))
            .body("age" equalTo("26"))
        ;
}
```
We are using the given(), when(), and then() methods provided by REST Assured.

Using **given()** we are setting the content type and request body

Using **when()** we are sending the put() request with endpoint URL and path variable(s)

Using **then()** we are verifying the response

### DELETE Method:
The DELETE method is used to remove information from the server.

To test the DELETE method, we need to delete user information using the given URL and user ID.

**URL:** https://example.com/api/users/{userId}

```java
@Test
void shouldDeleteUser() {
        given()
            .contentType("application/json")
        .when()
            .delete("https://example.com/api/users/{userId}",1)
        .then()
            .statusCode(200)
        ;
}
```
We are using the given(), when(), and then() methods provided by REST Assured.

Using **given()** we are setting the content type

Using **when()** we are sending delete() request with endpoint URL and path variable(s)

Using **then()** we are verifying the response


Use additional methods provided by REST Assured to test other aspects of the API, such as response headers, response body, and response time.

### Conclusion:
We have just explored the basic steps to get started with REST Assured testing. As you become more familiar with the library, you can use additional features such as authentication, request and response filters, and response specifications to create more advanced test cases.