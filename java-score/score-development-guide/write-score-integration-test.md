# Write SCORE integration test

This document explains how to write SCORE integration 

### Purpose

Understand how to write SCORE integration test

### Prerequisites

* [Score Overview](../overview.md)
* [Setup Goloop local Node](../../icon-2.0/goloop/get-started/build.md)
* General understanding of [testing in java.](https://junit.org/junit5/docs/current/user-guide/#writing-tests)

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

1. You can write and run the test method with @Test annotation.Test can be initialized and finalized 
setup and shutdown method.
2. Emulate ICON service for test.
   1. Initialize ICON service and confirm genesis block. 
   2. Create test wallets with a given icx valueOf.

3. Provide API for SCORE integration test.
   1. Request queries and send transactions to call SCORE APIs.
   2. Convert the requests and response values as Rpc.

### Examples

The example is  of HelloWorld SCORE whose source code can be found on java-score examples. 

**hello-world/src/main/java/com.iconloop.score.example/HelloWorld.java**
```java

package com.iconloop.score.example;

import score.Context;
import score.ObjectReader;
import score.ByteArrayObjectWriter;
import score.annotation.External;
import score.annotation.Payable;

public class HelloWorld {
    private final String name;

    public HelloWorld(String name) {
        this.name = name;
    }

    @External(readonly=true)
    public String name() {
        return name;
    }

    @External(readonly=true)
    public String getGreeting() {
        String msg = "Hello " + name + "!";
        Context.println(msg);
        return msg;
    }

    @Payable
    public void fallback() {
        // just receive incoming funds
    }

    @External
    public void testRlp() {
        var codec = "RLPn";
        var msg = "testRLP";

        ByteArrayObjectWriter w = Context.newByteArrayObjectWriter(codec);
        w.write(msg.getBytes());
        ObjectReader r = Context.newByteArrayObjectReader(codec, msg.getBytes());
    }
}

```

**hello-world/src/intTest/java/foundation.icon.test/cases/AppTestIT**

This contains different cases for the integration testing.

```java
package foundation.icon.test.cases;

import foundation.icon.icx.IconService;
import foundation.icon.icx.KeyWallet;
import foundation.icon.icx.data.TransactionResult;
import foundation.icon.icx.transport.http.HttpProvider;
import foundation.icon.test.*;
import foundation.icon.test.score.HelloWorldScore;
import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.Test;

import java.io.IOException;
import java.math.BigInteger;

import static foundation.icon.test.Env.LOG;

public class AppTestIT extends TestBase{
    private static TransactionHandler txHandler;
    private static KeyWallet ownerWallet;

    @BeforeAll
    static void setup() throws Exception {
        Env.Chain chain = Env.getDefaultChain();
        IconService iconService = new IconService(new HttpProvider(chain.getEndpointURL(3)));
        txHandler = new TransactionHandler(iconService,chain);

        //init wallets
        BigInteger amount =ICX.multiply(BigInteger.valueOf(100));
        ownerWallet = KeyWallet.create();
        txHandler.transfer(ownerWallet.getAddress(),amount);
        ensureIcxBalance(txHandler, ownerWallet.getAddress(), BigInteger.ZERO, amount);
    }

    @AfterAll
    static void shutdown() throws IOException {
        txHandler.refundAll(ownerWallet);
    }

    @Test
    void appHasAName() throws TransactionFailureException, IOException, ResultTimeoutException {
        HelloWorldScore helloWorldScore = HelloWorldScore.deploy(txHandler, ownerWallet);

        String name = helloWorldScore.name();
        Assertions.assertEquals("Alice",name);
    }

    @Test
    void appHasGreeting() throws TransactionFailureException, IOException, ResultTimeoutException {
        HelloWorldScore helloWorldScore = HelloWorldScore.deploy(txHandler, ownerWallet);
        String greeting = helloWorldScore.getGreeting();
        Assertions.assertEquals("Hello Alice!",greeting);
    }

    @Test
    void transfer() throws TransactionFailureException, IOException, ResultTimeoutException {
        HelloWorldScore helloWorldScore = HelloWorldScore.deploy(txHandler, ownerWallet);
        LOG.infoEntering("icx transfer to contract", "20 from owner wallet to score address");
        TransactionResult transfer = helloWorldScore.transfer(txHandler,ownerWallet,ICX.multiply(BigInteger.valueOf(20)));
        assertSuccess(transfer);
        LOG.infoExiting();
    }
}

```
**hello-world/intTest/score/HelloWorldScore**

This is the scenarios for the integration testing.
```java
package foundation.icon.test.score;

import foundation.icon.icx.Wallet;
import foundation.icon.icx.data.Bytes;
import foundation.icon.icx.data.TransactionResult;
import foundation.icon.icx.transport.jsonrpc.RpcObject;
import foundation.icon.icx.transport.jsonrpc.RpcValue;
import foundation.icon.test.ResultTimeoutException;
import foundation.icon.test.TransactionFailureException;
import foundation.icon.test.TransactionHandler;

import java.io.IOException;
import java.math.BigInteger;

import static foundation.icon.test.Env.LOG;

public class HelloWorldScore extends Score {

    public static HelloWorldScore deploy(TransactionHandler txHandler,
                                                Wallet wallet) throws TransactionFailureException, IOException, ResultTimeoutException {
        LOG.infoEntering("deploy","HelloWorld" );
        RpcObject params = new RpcObject.Builder()
                .put("name", new RpcValue("Alice"))
                .build();
        Score score = txHandler.deploy(wallet, getFilePath("hello-world"), params);
        LOG.info("scoreAddress = " + score.getAddress());
        LOG.infoExiting();
        return new HelloWorldScore(score);
    }

    public HelloWorldScore (Score other) {
        super(other);
    }

    public String name() throws IOException {
        return call("name", null).asString();
    }

    public String getGreeting() throws IOException {
        return call("getGreeting", null).asString();
    }

    public TransactionResult transfer(TransactionHandler txHandler, Wallet wallet, BigInteger value)
            throws IOException, ResultTimeoutException {

       Bytes txHash = txHandler.transfer(wallet,this.getAddress(),value);
       return getResult(txHash);
    }
}
```