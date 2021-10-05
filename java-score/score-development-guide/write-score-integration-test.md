# Write SCORE integration test

This document explains how to write SCORE integration 

### Purpose

Understand how to write SCORE integration test

### Prerequisites

* [Score Overview](../overview.md)
* [Setup Goloop local Node](../../icon-2.0/goloop/get-started/build.md)
* Basic concept of writing tests in java


### How to Write SCORE Integration Test Code

The SCORE integration test code works as follows.

1. Create an ICON JSON-RPC API request for the SCORE API you want to test. 
2. DEPLOY the SCORE to be tested. 
3. Call the task method to and get the result. 
4. Check the result. 
5. Shutdown the ICON JSON-RPc API request.

### Packages and modules

**ICON Java SDK**

You can create JSON-RPC API request using the ICON Java SDK

APIs can be called through IconService.IconService initialized as.
```java
// Creates an instance of IconService using the HTTP provider.
IconService iconService = new IconService(new HttpProvider("http://localhost:9000", 3));
```

Iconservice can be initialized with a custom HTTP client.

```java
OkHttpClient okHttpClient = new OkHttpClient.Builder()
.readTimeout(200, TimeUnit.MILLISECONDS)
.writeTimeout(600, TimeUnit.MILLISECONDS)
.build();

IconService iconService = new IconService(new HttpProvider(okHttpClient, "http://localhost:9000", 3));


```

**TestBase**

Every SCORE integration test class must inherit `TestBase`.

TestBase class provides the following functions:

1. Emulate ICON service for test
   1. Initialize ICON service and confirm genesis block. 
   2. Create test wallets with a given icx valueOf

2. Provide API for SCORE integration test.
   1. Request queries and send transactions to call SCORE APIs 
   2. Convert the requests and response values as Rpc

