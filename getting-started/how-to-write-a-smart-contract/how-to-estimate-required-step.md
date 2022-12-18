# How to estimate required STEP



It is important to set a proper `stepLimit` value in your transaction to make the submitted transaction executed successfully. This document explains how to systemically estimate the required Step.

### Intended Audience

Anyone who sends transactions to the ICON Network

### Purpose

To learn how to estimate the required Step for their transactions using ICON SDKs, T-Bears CLI or JSON-RPC API calls.

### Prerequisite

* Understand how ICON JSON-RPC APIs work.
* Need to know how to submit a transaction to the ICON Network using various tools.

### How-To

The mechanism of requesting Step estimation is very similar to sending transactions.

#### Request and Response

**Transaction Request**

The following example is a transaction request message that will invoke the `transfer` method on the IRC2 token contract.

```javascript
{
    "jsonrpc": "2.0",
    "method": "icx_sendTransaction",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxcced4d83a03098f5976b5ed339b0cab3e51ca1f9",
        "to": "cx65fc79aa09e867fd51d0c8361f42cb38bc1f08f3",
        "stepLimit": "0x30d40",
        "timestamp": "0x583f58f62eae0",
        "nid": "0x3",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "_to": "hx25cd4028f055457b8fbdae1dbd8b5fe465248e55",
                "_value": "0x8ac7230489e80000"
          }
        },
        "signature": "rBaH0U6y85y1CWp/JbalUbzLVGjtGYG+hut/G5o30vBxhWoxPYtSYBQu6X0Tak1SdcnlZSCJL7DeOeKmI4y+5wE="
    }
}
```

**Step Estimate Request**

The step estimation request is the same as the above example except the `stepLimit` and `signature`. The `stepLimit` and `signature` fields are unnecessary because the step estimation process does not verify the sender, nor limit the execution until the cumulative step reaches the `stepLimit`.

The following example is the Step estimation request for the above token transfer transaction. You can simply remove the `stepLimit` and `signature` fields from the original transaction request.

```javascript
{
    "jsonrpc": "2.0",
    "method": "debug_estimateStep",
    "id": 1234,
    "params": {
        "version": "0x3",
        "from": "hxcced4d83a03098f5976b5ed339b0cab3e51ca1f9",
        "to": "cx65fc79aa09e867fd51d0c8361f42cb38bc1f08f3",
        "timestamp": "0x583f58f62eae0",
        "nid": "0x3",
        "nonce": "0x1",
        "dataType": "call",
        "data": {
            "method": "transfer",
            "params": {
                "_to": "hx25cd4028f055457b8fbdae1dbd8b5fe465248e55",
                "_value": "0x8ac7230489e80000"
            }
        }
    }
}
```

**Response - Success**

If there are no exceptions, the response will return the estimated Step usage as follows.

```javascript
{
    "jsonrpc": "2.0",
    "id": 1234,
    "result": "0x23f8c"
}
```

**Response - Fail**

If there is an exception, the response will show the error message.

```javascript
{
    "jsonrpc": "2.0",
    "id": 1234,
    "error": {
        "code": -32602,
        "message": "JSON schema validation error: 'version' is a required property"
    }
}
```

#### JSON-RPC API call

Debug API Endpoint : :///api/debug/v3

You can make a step estimation request using curl from the CLI as follows. Note that in the below command example, `stepEstimationRequest.json` is the file that contains the request message.

```bash
$ curl -H "Content-Type: application/json" -d @stepEstimationRequest.json https://bicon.net.solidwallet.io/api/debug/v3
{"jsonrpc": "2.0", "result": "0x23f8c", "id": 1234}
```

#### Java

Below is a code snippet for Java. For the detailed ICON Java SDK usage guideline, please read [Java SDK](broken-reference).

```java
// make a raw transaction without the stepLimit
Transaction transaction = TransactionBuilder.newBuilder()
    .nid(networkId)
    .from(fromAddress)
    .to(toAddress)
    .nonce(BigInteger.valueOf(1))
    .call("transfer")
    .params(params)
    .build();

// get an estimated step value
BigInteger estimatedStep = iconService.estimateStep(transaction).execute();

// set some margin
BigInteger margin = BigInteger.valueOf(10000);

// make a signed transaction with the same raw transaction and the estimated step
SignedTransaction signedTransaction = new SignedTransaction(transaction, wallet, estimatedStep.add(margin));
Bytes txHash = iconService.sendTransaction(signedTransaction).execute();
...
```

#### Python

Step estimation code snippet for Python. For the detailed ICON Python SDK usage guideline, please read [Python SDK](broken-reference).

```python
# Generates a raw transaction without the stepLimit
transaction = TransactionBuilder()\
    .from_(wallet.get_address())\
    .to("cx00...02")\
    .value(150000000)\
    .nid(3)\
    .nonce(100)\
    .build()

# Returns an estimated step value
estimate_step = icon_service.estimate_step(transaction)

# Adds some margin to the estimated step 
step_limit = estimate_step + 10000

# Returns the signed transaction object having a signature with the same raw transaction and the estimated step
signed_transaction = SignedTransaction(transaction, wallet, step_limit)

# Sends the transaction
tx_hash = icon_service.send_transaction(signed_transaction)
```

### Summary

With the `debug_estimateStep` JSON-RPC API, developers can estimate how much Step will be required for a particular transaction under the current block state. Using various ICON SDKs, you can integrate the functionality in your DApp.

Please be aware that the returned value is just an **ESTIMATION**. The block state may not remain the same between the Step estimation and the actual transaction execution time. The required Step can be different if the block state changes.

### References

* [ICON JSON RPC](broken-reference)
* [Java SDK](broken-reference)
* [JavaScript SDK](broken-reference)
* [Python SDK](broken-reference)
* [Swift SDK](broken-reference)
