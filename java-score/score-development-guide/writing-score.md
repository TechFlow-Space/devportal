## Writing Score

The document presents how to write a SCORE, smart contract of the ICON network. Through the series of documents, 
you will learn from setting the workspace to deploying a SCORE.

SCORE is the platform for ICON’s smart contract. But it is also used to refer to the ICON’s smart contract itself.

### Intended Audience

The intended audience is the developers who have basic Java programming knowledge.

### Purpose

After reading this document, you will understand the structure of SCORE and learn the basic syntax of writing SCORE.

### Prerequisite

We assume you have already set up a local environment to deploy java score using gloop CLI library. While setting up 
the environment you would have to build the docker image and javaee-api.jar for local.
-	[Deploy Java Score](../../icon-2.0/java-score/tutorial.md)

### Creating a Workspace

The score is compiled the following the same structure as any other Java program. `gradle-javaee-plugin` is a Gradle 
plugin to automate the process of generating the optimized jar bundle. The generated jar bundle can be used for 
deployment to ICON networks that support the Java SCORE execution environment (a.k.a. JavaEE).

The `build.gradle` file contains the dependencies which contain the dependent packages for building the SCORE.
`optimizedJar` extends from Gradle’s `Jar` type i.e. all attributes and methods available on `Jar` are also available 
on optimizedJar. In optimizedJar `mainClassName` property is required to indicate the entry point for initial execution.
This property is included into the generated jar bundle with the `Main-Class` header in the manifest.

```groovy
    optimizedJar {
    mainClassName = 'com.iconloop.score.example.HelloWorld'
    }

```

The `deployJar` extension is used to deploy the optimized jar to local or remote ICON networks that support the Java 
SCORE execution environment.

```groovy
    deployJar {
    endpoints {
        local {
            uri = 'http://localhost:9082/api/v3'
            nid = 3
        }
    }
    keystore = './mykey.json'
    password = 'keypass'
    parameters {
        arg('name', 'Alice')
    }
    }
```


### Structure of Score

Every SCORE has a main class which is specified in the `build.gradle`. The main class should have more than one external 
method which is supposed to be invoked by transactions from EOA or other Smart Contracts. Transactions will result in 
the internal state update.

The following is the main class content for the [Hello world](https://github.com/icon-project/java-score-examples/blob/master/hello-world/src/main/java/com/iconloop/score/example/HelloWorld.java) score:

```java
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

```

Below is the comparison of Java to  Python Score Structure

| Name                | Python Score               | Java Score                  |
|---------------------|----------------------------|-----------------------------|
| External decorator  | @external                  | @External                   |
| -(readonly)         | @external(readonly=True)   | @External(readonly=true)    |
| Payable decorator   | @payable                   | @Payable|
| Eventlog decorator  | @eventlog                  | @EventLog                   |
| - (indexed)         | @eventlog(indexed=1)       | @EventLog(indexed=1)        |
| fallback signature  | def fallback               | void fallback()             |
| SCORE initialize    | override on_install method | define a public constructor |
| Default parameters  | native language support    | @Optional                   |

The details of the tables are discussed further down below.

### Implementing Score External Methods

Users can implement methods which are supposed to be involved from outside the blockchain. These methods can be 
decorated with `@External` and/or `@Payable`. Additionally, users can declare methods decorated with `@Eventlog`
which can be used to leave custom event logs in the TxResult.

**External Annotators (@External)**

This annotation can be used to indicate whether the method will be exposed externally or not.
Methods should be annotated as `@External` so that they can be called from outside the contract (EOA or another contract).
The annotated methods are registered on the exportable API list.
Any attempt to call a non-external method from outside the contract will fail.
If the `readonly` element is specified and its value is true, i.e. `@External (readonly =true)`, the method will have
read-only access to the state database.

**Payable Annotators (@Payable)**

This annotation can be used to indicate whether the method can receive ICX coins. External method annotated with 
`@Payable` can receive the incoming ICX coins designated in the transaction message and process further works for it. 
Users can get the value of transferred ICX coins by using `Context.getValue()`.
If ICX coins are passed to a non-payable method, that transaction would fail.

**Eventlog Annotators (@Eventlog)**

This annotation can be used to record logs in its TxResult as eventLogs.

If the value of an element, named `indexed`, is set, the designated number of parameters of the applied method 
declaration will be indexed in the order and included in the Bloom filter. Indexed parameters and non-indexed parameters
are separately stored in the TxResult. At most 3 parameters can be indexed, and the value of indexed cannot exceed 
the number of parameters. Possible data types for method parameters are `int, boolean, byte[], BigInteger, String, 
and Address`.

**Optional Annotators (@Optional)**

This annotation can be used to indicate whether the method parameter is optional. 
If a method parameter is annotated with this Optional, the parameter can be omitted 
in the transaction message. If optional parameters were omitted when the external method 
is called, the value of optional parameters would be their zero values.  The zero value is: 0 for numeric types, false 
for the boolean type, and null for Object types.

**Keep Annotators (@Keep)**

Denotes that the element should not be removed when the code is optimized by tool kit.


**fallback**

`fallback` is a special method that is invoked whenever the contract receives plain ICX coins without data.

```java
@Payable
public void fallback() {
    // just receive incoming funds
    }
```

However, if the `fallback` method is not annotated with `@Payable`, it would not be listed on the SCORE APIs and could 
not be called as well. The `fallback` method cannot be annotated with `@External`  (i.e., fallback method cannot be 
specified in the transaction message as a callee method)

### Built-In Properties

The score package has two classes using which you can interact with the score.
* [Address Class](address.md)
* [Context Class](context.md)


### Java Score Library for ICON Standard Tokens

By Including the package from [Maven Central](https://search.maven.org/search?q=g:com.github.sink772%20a:javaee-tokens)
in the dependency of `build.gradle` you can create ICON standard Tokens like IRC2 and IRC3. Using this you no longer
required to write the whole thing from scratch.

	implementation 'com.github.sink772:javaee-tokens:0.6.0'

You need to create an entry Java class to inherit the attributes and methods from the basic token classes.
The example below would be the simplest IRC2 token SCORE with a fixed supply.

```java
public class IRC2FixedSupply extends IRC2Basic {
    public IRC2FixedSupply(String _name, String _symbol) {
        super(_name, _symbol, 3);
        _mint(Context.getCaller(), BigInteger.valueOf(1000000));
    	}
	}
```

### Storing State Data

The state of SCORE should be stored in the state database in the blockchain. The state of SCORE simply means the 
values of member variables in the smart contract which are declared as VarDB, DictDB, and ArrayDB. The state of SCORE 
is changed only by the execution of transactions. All nodes execute transactions and change the state of the smart 
contract in their own state database independently. To be confirmed, the changed state should be agreed between the ⅔ 
of nodes after each execution of transactions, and this process is called consensus in the blockchain.

**VarDB(variable type)**

Variable type parameters can be  a readable and writable class. It holds one value.

**DictDB(key type, value type)**

A dictionary DB is a hash from key to value. Only values of the dictionary DB are recorded in the DB. Keys are not recorded.
Key type can be `String`, `byte array`, `Address`, `Byte`, `Short`, `Integer`, `Long`, `Character` or `BigInteger`.
Value Type can be readable and writable class

**ArrayDB(element type)**

An ArrayDB holds a sequence of values. Element type parameters is a readable and writable class.

**BranchDB(key type, value type)**

A branch DB is a hash from keys to sub-DBs.
Key type can be `String`, `byte array`, `Address`, `Byte`, `Short`, `Integer`, `Long`, `Character` or `BigInteger`.
Value type can be `VarDb`, `DictDB`, `ArrayDB` or `BranchDB`.

**ByteArrayObjectWriter**

```java
public interface ByteArrayObjectWriter extends ObjectWriter
```

**ObjectReader**

Interface for object read.

An object is readable if the object is a builtin object or its class has the following method.
```java
public static UserClass readObject(ObjectReader r)
```

A simple object is read by corresponding read method. The following is an example of reading simple objects.
```java
str = objectReader.readString();
i = objectReader.readInt();
```

A list is read by a beginList() call followed by calls for reading zero or more its elements followed by an end() call.

A map is read by a beginMap() call followed by calls for reading zero or more its elements followed by an end() call.

You can read a custom object indirectly if its class has the following method.
```java
public static UserClass readObject(ObjectReader r)

```
When you read a custom object of this class, the readObject method is called. In the method, you must read one 
equivalent builtin object. It is error to read no object or two or more objects. The method may read on list with 
multiple elements. If an exception is thrown during any read call, the reader becomes invalidated. An invalidated reader
fails any method.



**Object Writer**

Interface for object write.
An object is writable if the object is a builtin object or its class has the following method.
```java
public static void writeObject(ObjectWriter w, UserClass obj)
```

A builtin object is a simple object or a container object. A simple object is a Boolean, a Byte, a Short, a Character, 
a Integer, a Float, a Long, a Double, a BigInteger, a String, byte array or an Address. They are written by 
corresponding to write methods. The following is an example of writing simple objects.
```java
objectWriter.writeString("a string");
objectWriter.writeInt(0);
```
A container object is a list or a map. A container may have another container as its element.

A list has zero or more builtin objects as its elements. 
A list is written by a beginList(int) call followed by calls for writings of zero or more its elements followed by an 
end() call. For example, the following method writes a list of two elements.

```java
objectWriter.beginList(2);
    objectWriter.writeInt(0);
    objectWriter.beginList(0);
objectWriter.end();

```
A map has zero or more pairs of objects as its elements. A map is written by a beginMap(int) call followed by calls 
for writings of zero or more its elements followed by a end() call. For example, the following method writes a map of 
one element.
```java
objectWriter.beginMap(1);
    objectWriter.writeString("key");
    objectWriter.writeString("value");
objectWriter.end();

```
You can write a custom object indirectly if its class has the following method.
```java
public static void writeObject(ObjectWriter w, UserClass obj)
```

When you write a custom object, the writeObject method is called. In the method, you must write one equivalent builtin 
object. It is error to write no object or two or more objects. The method may write one list with multiple elements. 
For example, the following is error.

```java
public static void writeObject(ObjectWriter w, UserClass obj) {
w.writeString(obj.name);
w.writeString(obj.description);
}
```

Instead, write a list with multiple elements if you want to write multiple objects.
```java
public static void writeObject(ObjectWriter w, UserClass obj) {
w.beginList(2);
w.writeString(obj.name);
w.writeString(obj.description);
w.end();
}
```

You can write an object as a non-nullable type object or nullable type object. A non-nullable type object is always 
not null. A nullable object may be null or non-null. A write method writes an object as non-nullable type unless the 
document specifies the method writes a nullable type object. For example, write(String) writes non-nullable type 
object. writeNullable(Object) and writeNull() methods write a nullable type object. beginNullableList(int) begins a 
nullable list.

Nullable-ness shall be preserved during write and read. If a non-nullable object is written, the object shall be read 
as a non-nullable object and shall not be read as a nullable object later. If a nullable object is written, the object 
shall be read as a nullable object and shall not be read a non-nullable object later. For example, if an object is 
written as the following:
```java
Integer i = getInteger();
objectWriter.writeNullable(i);
```

It is error to read the object as the following:
```java
int i = objectReader.readInt();
```

Instead, the object shall be read as the following:
```java
Integer i = objectReader.readNullable(Integer.class);
```

If an exception is thrown during any write call, the writer becomes invalidated. An invalidated writer fails any method.


### Invoking Other Score Methods

One SCORE can invoke an external method of another SCORE using the following APIs.
```java
// [package score.Context]
public static Object call(Address targetAddress, String method, Object... params);

public static Object call(BigInteger value,Address targetAddress, String method, Object... params);

```

The following example is for calling `tokenFallback`.

```java
if (_to.isContract()) {
    Context.call(_to, "tokenFallback", _from, _value, dataBytes);
    }
```

### Exception Handling

**RevertedException**

Signals a failure of an inter-contract call. The construction constructs a new exception on calling.

**UserRevertedException**

Signals a manual reversion from a score. The construction constructs a new exception on calling.

**UserRevertException**

Signals failure of an External method. If an external method is called by an external transaction or an internal 
transaction and the method throws an exception of this class or subclass of this class during the execution, the 
transaction fails and the code returned by getCode() is used as the user failure code of the transaction. If the code 
is out of range, the code is clamped.
