# Write SCORE unit test

This document explains how to write SCORE unit-testing in java using JUnit 5.

## Purpose

Understand how to write a SCORE unit-test.

## Prerequisite

* SCORE overview
* General understanding of [testing in java.](https://junit.org/junit5/docs/current/user-guide/#writing-tests)

## How to Write SCORE Unit Test Code

SCORE unittest should inherit `TestBase`. Every test method should have `@Test` tag at first.

## Functions Provided by TestBase

* Instantiate SCORE 
  * Instantiate SCORE so that you can access the attribute and methods of SCORE like a general object.

* Get properties in SCORE
* Access the SCORE methods using the properties
* Use of HashMap for databases
  * Mock event logs
  * Using the [Mockito library](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
  you can mock events and verify the event logs have been called.
* Spy the methods
  * Spy annotations creates an instance of a class and tracks all the interactions within it. 
  Using this you can call methods of SCORE with the specified arguments.

## Methods

`TestBase` has 23 main methods.

**Getting SCORE instance**

* Deploy(owner, mainClass, Params)
  * Deploy an instance of SCORE 
  * Parameters 
    * **owner**: Address to set as owner of SCORE 
    * **mainClass**: Class which initiates the SCORE development 
    * **params**: parameter required at first on installing the SCORE
    
**Getting SCORE properties**

* getOwner()
  * get Address of the owner the score id deployed to.
* getInstance()
  * returns the instance of the block
* getHeight()
  * returns height of the block
* getTimestamp()
  * returns the timestamp of the block
* getBlock()
  * returns block instance 
* getOrigin()
  * get Origin address of the score.

**Patching calls**

* call (from, value, targetAddress, method, params )
  * called by Wallet Address 
  * Parameters 
    * **from**: caller which initiated the method 
    * **value**: value 
    * **targetAddress**: address of SCORE having method to be called 
    * **method**: method to be called 
    * **params**: parameters to call the methods

* call (caller, value, targetAddress, method, params)
  * called by Contract 
  * Parameters 
    * **caller**: Class which calls the method 
    * **value**: value 
    * **targetAddress**: address of SCORE having method to be called 
    * **method**: method to be called 
    * **params**: parameters to call the methods

**Utils**

* transfer()
  * transfer icx to the address
  * Parameters
    * **from** : sender
    * **to** : receiver
    * **value**: amount to transfer
* getCaller()
  * get address of the current score caller
* createAccount()
  * creates an Account with 0 icx
* createAccount(initialIcx)
  * creates an Account with passed initial icx
  * Parameter 
    * **initialIcx**: initial icx amount for the new account
* getAddress()
  * get address of current score
* getScoreFromClass(caller)
  * Get information on whether the score is from the class called
  * Parameter
    * **caller**: Class the score is being called by
* getScoreFromAddress(target)
  * Get information on whether the score if in the target address or not
  * Parameter
    * **target** : Address of the score
* putStorage(key,value)
  * Store the Address to the hashmap in blockchain
  * Parameters
    * **key**: class of the score
    * **value** : score instance
* getStorage(key)
  * get Address form the storage 
  * Parameters 
    * **key**: class of the score to get the Address of
* isReadonly()
  * returns boolean value for readonly of the method
* getValue()
  * returns the value of method


