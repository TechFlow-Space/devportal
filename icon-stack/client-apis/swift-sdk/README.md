# Swift SDK

###

ICON supports SDK for 3rd party or user services development. You can integrate ICON SDK for your project and utilize the ICON’s functionality. This document provides you with information about API specification.

### `class` ICONService

`ICONService` is a class which provides APIs to communicate with ICON nodes. It enables you to easily use [ICON JSON-RPC APIs](broken-reference) (version 3). All methods of `IconService` returns a `Request<T, ICError>` instance.

#### Query Execution

`Request` are executed as **synchronized** and **asynchronized**.

**Synchronized execution**

To execute the request synchronously, you need to run `execute()` function of `Request`.

```swift
public func execute() -> Result<T, ICError>
```

**Returns**

`Result<T, ICError>`

**Example**

```swift
// Synchronized request execution
let response = iconService.getLastBlock().execute()

switch response {
case .success(let block):
    print(block)
case .failure(let error):
    print(error.errorDescription)
}
```

**Asynchronized execution**

To execute the request asynchronously, you need to run `async()` function of `Request`.

```swift
public func async(_ completion: @escaping (Result<T, ICError>) -> Void)
```

**Returns**

`async()` is a closure. Passing `Result<T, ICError>`.

**Example**

```swift
// Asynchronized request execution
iconService.getLastBlock().async { (result) in
    switch result {
    case .success(let block):
      print(block)
    case .failure(let error):
      print(err.errorDescription)
    }
}
```

#### Transaction Execution

`Transaction` class is used to make a transaction instance. There are 3 types of Transaction class.

| Class                                             | Description                                                                                                         |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| [Transaction](./#class-transaction)               | A class for Transaction instance which is for sending ICX.                                                          |
| [CallTransaction](./#class-calltransaction)       | A class for CallTransaction instance which is for SCORE function call. CallTransaction extends `Transaction` class. |
| [MessageTransaction](./#class-messagetransaction) | A class for MessageTransaction instance which is for transfer message. Extends `Transaction` class.                 |

Request is executed as **Synchronized** or **Asynchronized** like a querying request.

**Example**

```swift
// Synchronized request
let response = iconService.sendTransaction(signedTransaction: signed).execute()

switch response {
case .success(let result): // result == String
    print(result)
case .error(let error):
    print(error)
}

// Asynchronized request
let request = iconService.sendTransaction(signedTransaction: signed)
request.async { (result) in
    switch result {
    case .success(let result): // result == String
        print(result)
    case .failure(let error):
        print(error)
    }
}
```

#### Initialize

```swift
init(provider: String, nid: String)
```

**Parameters**

| Parameter | Type     | Description    |
| --------- | -------- | -------------- |
| provider  | `String` | ICON node url. |
| nid       | `String` | Network ID.    |

For more details of node URL and network id, see [ICON Network](broken-reference) document.

**Example**

```swift
// ICON Mainnet
let iconService = ICONService(provider: "https://ctz.solidwallet.io/api/v3", nid: "0x1")
```

#### `func` getLastBlock

Get the last block information.

```swift
func getLastBlock() -> Request<Response.Block>
```

**Parameters**

None

**Returns**

`Request` - Request instance for `icx_getLastBlock` JSON-RPC API request. If the execution is successful, returns `Result<Response.Block, ICError>`.

**Example**

```swift
// Returns last block
let response = iconService.getLastBlock().execute()

switch response {
case .success(let result): // result == Response.Block
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` getBlock

Get the block information.

```swift
// Get block information by height.
func getBlock(height: UInt64) -> Request<Response.Block>

// Get block information by hash.
func getBlock(hash: String) -> Request<Response.Block>
```

**Parameters**

| Parameter | Type     | Description           |
| --------- | -------- | --------------------- |
| height    | `UInt64` | Height value of block |
| hash      | `String` | Hash value of block   |

**Returns**

`Request` - Request instance for `icx_getBlockByHeight` | `icx_getBlockByHash`. If the execution is successful, returns `Result<Response.Block, ICError>`.

**Example**

```swift
// Returns block information by block height
let heightBlock = iconService.getBlock(height: 10532).execute()

// Returns block information by block hash
let hashBlock = iconService.getBlock(hash: "0xdb310dd653b2573fd673ccc7489477a0b697333f77b3cb34a940db67b994fd95").execute()

switch heightBlock {
case .success(let block): // result == Response.Block
    print(block)
case .error(let error):
    print(error)
}
```

#### `func` call

Calls an external function of SCORE API.

```swift
func call<T>(_ call: Call<T>) -> Request<T>
```

**Parameters**

| Parameter | Type   | Description                |
| --------- | ------ | -------------------------- |
| call      | `Call` | an instance of Call class. |

**Returns**

`Request<T>` - Request instance for `icx_call` JSON-RPC request. If the execution is successful, returns `Result<T, ICError>`.

**Example**

In `Call` generic value, input the type of function call. The generic value should follow Codable protocol.

For example, the type of SCORE function call is `String`, input `String` like this.

```swift
// Create Call instance 
let call = Call<String>(from: wallet.address, to: "cxf9148db4f8ec78823a50cb06c4fed83660af38d0", method: "tokenOwner", params: nil)

let request: Request<String> = iconService.call(call)
let response: Result<String, ICError> = request.execute()

switch response {
case .success(let owner):
     print(owner) // hx70b591822468415e86fb9ba1354214902ea76196
case .failure(let error):
    print(error)
}
```

If SCORE call result is hex `String`, you can choice return types `String` or `BigUInt`.

Using `Call<String>`, return an original hex String value.

Using `Call<BigUInt>` return converted BigUInt value.

```swift
let ownerAddress: String = "hx07abc7a5b8a4941fc0b6930c88b462995acf929b"
let scoreAddress: String = "cxf9148db4f8ec78823a50cb06c4fed83660af38d0"

// Using `String`. Return hex String.
let call = Call<String>(from: ownerAddress, to: scoreAddress, method: "balanceOf", params: ["_owner": ownerAddress])
let request: Request<String> = iconService.call(call)
let response: Result<String, ICError> = request.execute()

switch response {
case .success(let balance):
     print("balance: \(balance)") // "balance: 0x11e468ef422cc60031ef"
case .failure(let error):
    print(error)
}


// Using `BigUInt`. Return BigUInt.
let call = Call<BigUInt>(from: ownerAddress, to: scoreAddress, method: "balanceOf", params: ["_owner": ownerAddress])
let request: Request<BigUInt> = iconService.call(call)
let response: Result<BigUInt, ICError> = request.execute()

switch response {
case .success(let balance):
     print("balance: \(balance)") // "balance: 84493649192649192649199"
case .failure(let error):
    print(error)
}
```

#### `func` getBalance

Get the balance of the address.

```swift
func getBalance(address: String) -> Request<BigUInt>
```

**Parameters**

| Parameter | Type     | Description    |
| --------- | -------- | -------------- |
| address   | `String` | An EOA Address |

**Returns**

`Request` - Request instance for `icx_getBalance` JSON-RPC API request. If the execution is successful, returns `Result<BigUInt, ICError>`.

**Example**

```swift
// Returns the balance of specific address
let response = iconService.getBalance(address: "hx9d8a8376e7db9f00478feb9a46f44f0d051aab57").execute()

switch response {
case .success(let result): // result == BigUInt
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` getScoreAPI

Get the SCORE API list.

```swift
func getScoreAPI(scoreAddress: String) -> Request<[Response.ScoreAPI]>
```

**Parameters**

| Parameter    | Type     | Description     |
| ------------ | -------- | --------------- |
| scoreAddress | `String` | A SCORE address |

**Returns**

`Request` - Request instance for `icx_getScoreApi` JSON-RPC API request. If the execution is successful, returns `Result<[Response.ScoreAPI], ICError>`.

**Example**

```swift
// Returns the SCORE API list
let response = iconService.getScoreAPI(scoreAddress: "cx0000000000000000000000000000000000000001").execute()

switch response {
case .success(let result): // result == [Response.ScoreAPI]
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` getTotalSupply

get the total number of issued coins.

```swift
func getTotalSupply() -> Request<BigUInt>
```

**Parameters**

None

**Returns**

`Request` - Request instance for `icx_getTotalSupply` JSON-RPC API request. If the execution is successful, returns `Result<BigUInt, ICError>`.

**Example**

```swift
// Returns total value of issued coins
let response = iconService.getTotalSupply().execute()

switch response {
case .success(let result): // result == BigUInt
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` getTransaction

Get the transaction information.

```swift
func getTransaction(hash: String) -> Request<Response.TransactionByHashResult>
```

**Parameters**

| Parameter | Type     | Description               |
| --------- | -------- | ------------------------- |
| hash      | `String` | Hash value of transaction |

**Returns**

`Request` - Request instnace for `icx_getTransactionByHash` JSON-RPC API request. If the execution is successful, returns `Result<Response.TransactionByHashResult, ICError>`.

**Example**

```swift
// Returns transaction information
let response = iconService.getTransaction(hash: "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238").execute()

switch response {
case .success(let result): // result == Response.TransactionByHashResult
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` getTransactionResult

Get the transaction result information.

```swift
func getTransactionResult(hash: String) -> Request<Response.TransactionResult>
```

**Parameters**

| Parameter | Type     | Description         |
| --------- | -------- | ------------------- |
| hash      | `String` | A transaction hash. |

**Returns**

`Request` - Request instance for `icx_getTransactionResult` JSON-RPC API request. If the execution is successful, returns `Result<Response.TransactionResult, ICError>`.

**Example**

```swift
// Returns transaction result information
let response = iconService.getTransactionResult(hash: "0xb903239f8543d04b5dc1ba6579132b143087c68db1b2168786408fcbce568238").execute()

switch response {
case .success(let result): // result == Response.TransactionResult
    print(result)
case .error(let error):
    print(error)
}
```

#### `func` sendTransaction

Send a transaction that changes the state of address. Request is executed as **Synchronized** or **Asynchronized** like a querying request.

```swift
func sendTransaction(signedTransaction: SignedTransaction) -> Request<String>
```

**Parameters**

| Parameter         | Type                | Description                                                           |
| ----------------- | ------------------- | --------------------------------------------------------------------- |
| signedTransaction | `SignedTransaction` | an instance of [SignedTransaction](./#class-signedtransaction) class. |

**Returns**

`Request` - Request instance for `icx_sendTransaction` JSON-RPC request. If the execution is successful, returns `Result<String, ICError>`.

**Example**

```swift
// Synchronized request
let response = iconService.sendTransaction(signedTransaction: signed).execute()

switch response {
case .success(let result): // result == String
    print(result)
case .error(let error):
    print(error)
}

// Asynchronized request
let request = iconService.sendTransaction(signedTransaction: signed)
request.async { (result) in
    switch result {
    case .success(let result): // result == String
        print(result)
    case .failure(let error):
        print(error)
    }
}
```

### `class` Wallet

```swift
class Wallet
```

A class which provides EOA functions. It enables you to create, transform to Keystore or load wallet from Keystore.

#### Initialize

```swift
init(privateKey: PrivateKey?)
```

**Parameters**

| Parameter  | Type         | Description                                                             |
| ---------- | ------------ | ----------------------------------------------------------------------- |
| privateKey | `PrivateKey` | If privateKey is null, it generates new private key while initializing. |

**Returns**

`Wallet` - Wallet instance.

**Example**

```swift
// Create new wallet
let wallet = Wallet(prviateKey: nil)    // will generate private key

// Imports from exist private key
let privateKey = PrivateKey(hexData: privateKeyData)
let wallet = Wallet(privateKey: privateKey)
```

```swift
// Loading wallets and storing the Keystore

// Save wallet keystore.
let wallet = Wallet(privateKey: nil)
do {
    try wallet.generateKeystore(password: "YOUR_WALLET_PASSWORD")
    try wallet.save(filepath: "YOUR_STORAGE_PATH")
} catch {
    // handle errors
}

// Load a wallet from the keystore.
do {
    let jsonData: Data = try Data(contentsOf: "YOUR_KEYSTORE_PATH")
    let decoder = JSONDecoder()
    let keystore = try decoder.decoder(Keystore.self, from: jsonData)
    let wallet = Wallet(keystore: keystore, password: "YOUR_WALLET_PASSWORD")
} catch {
    // handle errors
}
```

#### `func` getSignature

Generate signature string by signing transaction data.

```swift
func getSignature(data: Data) throws -> String
```

**Parameter**

| Parameter | Type   | Description                      |
| --------- | ------ | -------------------------------- |
| data      | `Data` | the serialized transaction data. |

**Returns**

`String` - A signature string.(**Base64 encoded**)

**Example**

```swift
let wallet = Wallet(privateKey: nil)
do {
    let signature = try wallet.getSignature(data: toBeSigned)
    print("signature - \(signature)")
} catch {
    // handle error
}
```

#### Properties

| Property | Type     | Description                         |
| -------- | -------- | ----------------------------------- |
| address  | `String` | Return wallet's address (read-only) |

### `class` Transaction

```swift
class Transaction
```

`Transaction` is a class representing a transaction data used for sending ICX.

#### Initialize

```swift
// Create empty transaction object
init()
// Create transaction with data
convenience init(from: String, to: String, stepLimit: BigUInt, nid: String, value: BigUInt, nonce: String, dataType: String? = nil, data: Any? = nil)
```

**Parameters**

| Parameter | Description                                                                  |
| --------- | ---------------------------------------------------------------------------- |
| from      | An EOA address that creates the transaction.                                 |
| to        | An EOA address to receive coins or SCORE address to execute the transaction. |
| stepLimit | Amounts of Step limit.                                                       |
| nid       | A network ID.                                                                |
| value     | Sending the amount of ICX in loop unit.                                      |
| nonce     | An arbitrary number used to prevent transaction hash collision.              |

#### `func` from

Setter for `from` property.

```swift
func from(_ from: String) -> Self
```

**Parameters**

| Parameter | Type     | Description                                  |
| --------- | -------- | -------------------------------------------- |
| from      | `String` | An EOA address that creates the transaction. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
// Set from property
let transaction = Transaction()
        .from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
```

#### `func` to

Setter for `to` property.

```swift
func to(_ to: String) -> Self
```

**Paramters**

| Paramter | Type     | Description              |
| -------- | -------- | ------------------------ |
| to       | `String` | An EOA or SCORE address. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
        .to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
```

#### `func` stepLimit

Setter for `setpLimit` property.

```swift
func stepLimit(_ limit: BigUInt) -> Self
```

**Parameters**

| Parameter | Type      | Description           |
| --------- | --------- | --------------------- |
| limit     | `BigUInt` | Amounts of Step limit |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
        .stepLimit(BigUInt(1000000))
```

#### `func` nid

Setter for `nid` property.

```swift
func nid(_ nid: String) -> Self
```

**Parameters**

| Parameter | Type     | Description                                                                                                                                       |
| --------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| nid       | `String` | A network ID. (Mainnet = "0x1", Testnet = "0x2", [etc](https://github.com/icon-project/icon-project.github.io/blob/master/docs/icon\_network.md)) |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
        .nid("0x1")
```

#### `func` value

Setter for `value` property.

```swift
func value(_ value: BigUInt) -> Self
```

**Parameters**

| Parameter | Type      | Description                                             |
| --------- | --------- | ------------------------------------------------------- |
| value     | `BigUInt` | Sending amount of ICX in loop unit. (1 ICX = 1018 loop) |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
    .value(BigUInt(15000000))
```

#### `func` nonce

Setter for `nonce` property.

```swift
func nonce(_ nonce: String) -> Self
```

**Parameters**

| Parameter | Type     | Description    |
| --------- | -------- | -------------- |
| nonce     | `String` | A nonce value. |

**Returns**

Returns `Transaction` itself.

**Example**

```swift
let transaction = Transaction()
    .nonce("0x1")
```

```swift
// Creating transaction instance for Sending ICX.
let transaction = Transaction()
    .from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
    .to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .nid("0x1")
    .nonce("0x1")
```

### `class` CallTransaction

```swift
class CallTransaction: Transaction
```

`CallTransaction` class is used for invoking a state-transition function of SCORE. It extends `Transaction` class, so instance parameters and methods of the class are mostly identical to `Transaction` class, except for the following:

**Parameters**

| Parameters | Description                   |
| ---------- | ----------------------------- |
| method     | The method name of SCORE API. |
| params     | The input params for method.  |

For details of extended parameters and methods, see [Transaction](./#class-transaction) section.

#### `func` method

Transaction for invoking a _state-transition_ function of SCORE.

_Transaction parameter `dataType` will be fixed with `call`_

```swift
func method(_ method: String) -> Self
```

**Parameters**

| Parameter | Type     | Description                   |
| --------- | -------- | ----------------------------- |
| method    | `String` | the method name of SCORE API. |

**Returns**

Returns `CallTransaction` itself.

**params**

The input parameters of the SCORE method that will be executed by `call` function.

```swift
func params(_ params: [String: Any]) -> Self
```

**Parameter**

| Parameter | Type                      | Description                       |
| --------- | ------------------------- | --------------------------------- |
| params    | `Dictionary<String: Any>` | Input parameters for call method. |

**Returns**

Returns `CallTransaction` itself.

**Example**

```swift
// Creating transaction instance for SCORE function call
let call = CallTransaction()
    .from(wallet.address)
    .to("cx000...001")
    .stepLimit(BigUInt(1000000))
    .nid(self.iconService.nid)
    .nonce("0x1")
    .method("transfer")
    .params(["_to": to, "_value": "0x1234"])
```

### `class` MessageTransaction

```swift
class MessageTransaction: Transaction
```

`MessageTransaction` class is used for sending message data. It extends `Transaction` class, so instance parameters and methods of the class are mostly identical to `Transaction` class, except for the following:

**Parameters**

| Parameters | Description        |
| ---------- | ------------------ |
| message    | A message to send. |

For details of extended parameters and methods, see [Transaction](./#class-transaction) section.

#### `func` message

Send messages.

_Transaction parameter `dataType` will be fixed with `message`._

```swift
func message(_ message: String) -> Self
```

**Parameters**

| Parameter | Type     | Description       |
| --------- | -------- | ----------------- |
| message   | `String` | A message String. |

**Returns**

Returns `MessageTransaction` itself.

**Example**

```swift
// Creating transaction instance for transfering message.
let messageTransaction = MessageTransaction()
    .from("hx9043346dbaa72bca42ecec6b6e22845a4047426d")
    .to("hx2e26d96bd7f1f46aac030725d1e302cf91420458")
    .value(BigUInt(15000000))
    .stepLimit(BigUInt(1000000))
    .nonce("0x1")
    .nid("0x1")
    .message("Hello, ICON!")
```

### `class` SignedTransaction

`SignedTransaction` is a class to make a signed transaction.

#### Initialize

```swift
init(transaction: Transaction, privateKey: PrivateKey)
```

**Parameters**

| Parameter   | Type          | Description                        |
| ----------- | ------------- | ---------------------------------- |
| transaction | `Transaction` | A transaction that will be signed. |
| privateKey  | `PrivateKey`  | A privateKey.                      |

**Example**

```swift
let signed = try SignedTransaction(transaction: transaction, privateKey: yourPrivateKey)
```

### ICError

There are 5 types of errors.

* **emptyKeystore** - Keystore is empty.
* **invalid**
  * missing(parameter: JSONParameterKey) - Missing JSON parameter.
  * malformedKeystore - Keystore data malformed.
  * wrongPassword - Wrong password.
* **fail**
  * sign - Failed to sign.
  * parsing - Failed to parse.
  * decrypt - Failed to decrypt.
  * convert - Failed to convert to URL or data.
* **error(error: Error)**
* **message(error: String**) - [JSON RPC Error Messages](broken-reference)

### Converter Functions

ICONKit supports converter functions.

#### convert()

Convert ICX or gLoop to loop.

> 1 ICX = 109 gLoop = 1018 loop

**Paramters**

| Parameters | Type   | Description                                                              |
| ---------- | ------ | ------------------------------------------------------------------------ |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is `.icx`. |

**Returns**

`BigUInt` - The value that converted to loop.

**Example**

```swift
let balance: BigUInt = 100

// Convert ICX to loop.
let ICXToLoop: BigUInt = balance.convert() // 100000000000000000000

// Convert gLoop to loop
let gLoopToLoop: BigUInt = balance.convert(unit: .gLoop) // 100000000000
```

#### toHexString()

Convert `BigUInt` value to hex `String`.

```swift
public func toHexString(unit: Unit = .loop) -> String
```

**Parameters**

| Parameters | Type   | Description                                                               |
| ---------- | ------ | ------------------------------------------------------------------------- |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is `.loop`. |

**Returns**

`String` - The value converted to a hex String.

**Example**

```swift
// Convert `BigUInt` value to hex `String`.
let ICXToLoop: BigUInt = 100000000000000000000
let hexString: String = ICXToLoop.toHexString() // 0x56bc75e2d63100000
```

#### hexToBigUInt()

Convert hex `String` to `BigUInt`.

```swift
public func hexToBigUInt(unit: Unit = .loop) -> BigUInt?
```

**Parameters**

| Parameters | Type   | Description                                                               |
| ---------- | ------ | ------------------------------------------------------------------------- |
| unit       | `Unit` | The unit of value( `.icx`, `.gLoop`, `.loop` ). Default value is `.loop`. |

**Returns**

`BigUInt` - The value that converted to hex String.

If the conversion is failed, return `nil`.

**Example**

```swift
// Convert hex `String` to `BigUInt`.
let hexString: String = "0x56bc75e2d63100000"
let hexBigUInt: BigUInt = hexString.hexToBigUInt()! // 100000000000000000000
```

### References

* [ICON JSON-RPC API v3](broken-reference)
* [ICON Network](broken-reference)

