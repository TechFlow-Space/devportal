# Write SCORE unit test

This document explains how to write SCORE unit-testing in java using JUnit 5.

## Purpose

Understand how to write a SCORE unit-test.

## Prerequisite

* [Score Overview](../overview.md)
* General understanding of [testing in java.](https://junit.org/junit5/docs/current/user-guide/#writing-tests)

## How to Write SCORE Unit Test Code

SCORE unittest should inherit `TestBase`. Every test method should have `@Test` annotation at first.

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

### Examples

The example is  of HelloWorld SCORE whose source code can be found on java-score examples. 

**sample-crowdsale/src/main/java/com.iconloop.score.example/SampleCrowdsale.java**

```java

package com.iconloop.score.example;

import score.Address;
import score.Context;
import score.DictDB;
import score.VarDB;
import score.annotation.EventLog;
import score.annotation.External;
import score.annotation.Payable;

import java.math.BigInteger;

public class SampleCrowdsale
{
  private static final BigInteger ONE_ICX = new BigInteger("1000000000000000000");
  private final Address beneficiary;
  private final Address tokenScore;
  private final BigInteger fundingGoal;
  private final long deadline;
  private boolean fundingGoalReached;
  private boolean crowdsaleClosed;
  private final DictDB<Address, BigInteger> balances;
  private final VarDB<BigInteger> amountRaised;

  public SampleCrowdsale(BigInteger _fundingGoalInIcx, Address _tokenScore, BigInteger _durationInBlocks) {
    // some basic requirements
    Context.require(_fundingGoalInIcx.compareTo(BigInteger.ZERO) >= 0);
    Context.require(_durationInBlocks.compareTo(BigInteger.ZERO) >= 0);

    this.beneficiary = Context.getCaller();
    this.fundingGoal = ONE_ICX.multiply(_fundingGoalInIcx);
    this.tokenScore = _tokenScore;
    this.deadline = Context.getBlockHeight() + _durationInBlocks.longValue();

    this.fundingGoalReached = false;
    this.crowdsaleClosed = true; // Crowdsale closed by default

    this.balances = Context.newDictDB("balances", BigInteger.class);
    this.amountRaised = Context.newVarDB("amountRaised", BigInteger.class);
  }

  @External(readonly=true)
  public String name() {
    return "SampleCrowdsale";
  }

  /*
   * Receives initial tokens to reward to the contributors.
   */
  @External
  public void tokenFallback(Address _from, BigInteger _value, byte[] _data) {
    // check if the caller is a token SCORE address that this SCORE is interested in
    Context.require(Context.getCaller().equals(this.tokenScore));

    // depositing tokens can only be done by owner
    Context.require(Context.getOwner().equals(_from));

    // value should be greater than zero
    Context.require(_value.compareTo(BigInteger.ZERO) >= 0);

    // start Crowdsale hereafter
    Context.require(this.crowdsaleClosed);
    this.crowdsaleClosed = false;
    // emit eventlog
    CrowdsaleStarted(this.fundingGoal, this.deadline);
  }

  /*
   * Called when anyone sends funds to the SCORE and that funds would be regarded as a contribution.
   */
  @Payable
  public void fallback() {
    // check if the crowdsale is closed
    Context.require(!this.crowdsaleClosed);

    Address _from = Context.getCaller();
    BigInteger _value = Context.getValue();
    Context.require(_value.compareTo(BigInteger.ZERO) > 0);

    // accept the contribution
    BigInteger fromBalance = safeGetBalance(_from);
    this.balances.set(_from, fromBalance.add(_value));

    // increase the total amount of funding
    BigInteger amountRaised = safeGetAmountRaised();
    this.amountRaised.set(amountRaised.add(_value));

    // give tokens to the contributor as a reward
    byte[] _data = "called from Crowdsale".getBytes();
    Context.call(this.tokenScore, "transfer", _from, _value, _data);
    // emit eventlog
    FundTransfer(_from, _value, true);
  }
  
  /*
   * Checks if the goal has been reached and ends the campaign.
   */
  @External
  public void checkGoalReached() {
    if (afterDeadline()) {
      if (!this.crowdsaleClosed) {
        this.crowdsaleClosed = true;
        // emit eventlog
        CrowdsaleEnded();
      }
      BigInteger amountRaised = safeGetAmountRaised();
      if (amountRaised.compareTo(this.fundingGoal) >= 0) {
        this.fundingGoalReached = true;
        // emit eventlog
        GoalReached(this.beneficiary, amountRaised);
      }
    }
  }

  /*
   * Withdraws the funds safely.
   *
   *  - If the funding goal has been reached, sends the entire amount to the beneficiary.
   *  - If the goal was not reached, each contributor can withdraw the amount they contributed.
   */
  @External
  public void safeWithdrawal() {
    if (afterDeadline()) {
      Address _from = Context.getCaller();

      // each contributor can withdraw the amount they contributed if the goal was not reached
      if (!this.fundingGoalReached) {
        BigInteger amount = safeGetBalance(_from);
        if (amount.compareTo(BigInteger.ZERO) > 0) {
          // set their balance to ZERO first before transferring the amount to prevent reentrancy attack
          this.balances.set(_from, BigInteger.ZERO);
          // transfer the icx back to them
          Context.transfer(_from, amount);
          // emit eventlog
          FundTransfer(_from, amount, false);
        }
      }

      // owner can withdraw the contribution since the sales target has been met.
      if (this.fundingGoalReached && this.beneficiary.equals(_from)) {
        BigInteger amountRaised = safeGetAmountRaised();
        if (amountRaised.compareTo(BigInteger.ZERO) > 0) {
          // transfer the funds to beneficiary
          Context.transfer(this.beneficiary, amountRaised);
          // emit eventlog
          FundTransfer(this.beneficiary, amountRaised, false);
          // reset amountRaised
          this.amountRaised.set(BigInteger.ZERO);
        }
      }
    }
  }

  private BigInteger safeGetBalance(Address owner) {
    return this.balances.getOrDefault(owner, BigInteger.ZERO);
  }

  private BigInteger safeGetAmountRaised() {
    return this.amountRaised.getOrDefault(BigInteger.ZERO);
  }

  private boolean afterDeadline() {
    // checks if it has been reached to the deadline block
    return Context.getBlockHeight() >= this.deadline;
  }

  @EventLog
  protected void CrowdsaleStarted(BigInteger fundingGoal, long deadline) {}

  @EventLog
  protected void CrowdsaleEnded() {}

  @EventLog(indexed=3)
  protected void FundTransfer(Address backer, BigInteger amount, boolean isContribution) {}

  @EventLog(indexed=2)
  protected void GoalReached(Address recipient, BigInteger totalAmountRaised) {}
}

```

**sample-crowdsale/src/test/java/com.iconloop.score.example/SampleCrowdsaleTest**

```java

package com.iconloop.score.example;

import com.iconloop.score.test.Account;
import com.iconloop.score.test.Score;
import com.iconloop.score.test.ServiceManager;
import com.iconloop.score.test.TestBase;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import java.math.BigInteger;

import static java.math.BigInteger.TEN;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.ArgumentMatchers.anyLong;
import static org.mockito.ArgumentMatchers.eq;
import static org.mockito.Mockito.never;
import static org.mockito.Mockito.spy;
import static org.mockito.Mockito.verify;

class SampleCrowdsaleTest extends TestBase {
  // sample-token
  private static final String name = "MySampleToken";
  private static final String symbol = "MST";
  private static final int decimals = 18;
  private static final BigInteger initialSupply = BigInteger.valueOf(1000);
  private static final BigInteger totalSupply = initialSupply.multiply(TEN.pow(decimals));

  // sample-crowdsale
  private static final BigInteger fundingGoalInICX = BigInteger.valueOf(100);
  private static final BigInteger durationInBlocks = BigInteger.valueOf(32);

  private static final ServiceManager sm = getServiceManager();
  private static final Account owner = sm.createAccount();
  private Score tokenScore;
  private Score crowdsaleScore;

  private SampleCrowdsale crowdsaleSpy;
  private final byte[] startCrowdsaleBytes = "start crowdsale".getBytes();

  @BeforeEach
  public void setup() throws Exception {
    // deploy token and crowdsale scores
    tokenScore = sm.deploy(owner, IRC2BasicToken.class,
            name, symbol, decimals, initialSupply);
    crowdsaleScore = sm.deploy(owner, SampleCrowdsale.class,
            fundingGoalInICX, tokenScore.getAddress(), durationInBlocks);

    // setup spy object against the crowdsale object
    crowdsaleSpy = (SampleCrowdsale) spy(crowdsaleScore.getInstance());
    crowdsaleScore.setInstance(crowdsaleSpy);
  }

  private void startCrowdsale() {
    // transfer all tokens to crowdsale score
    tokenScore.invoke(owner, "transfer", crowdsaleScore.getAddress(), totalSupply, startCrowdsaleBytes);
  }

  @Test
  void tokenFallback() {
    startCrowdsale();
    // verify
    verify(crowdsaleSpy).tokenFallback(owner.getAddress(), totalSupply, startCrowdsaleBytes);
    verify(crowdsaleSpy).CrowdsaleStarted(eq(ICX.multiply(fundingGoalInICX)), anyLong());
    assertEquals(totalSupply, tokenScore.call("balanceOf", crowdsaleScore.getAddress()));
  }

  @Test
  void fallback_crowdsaleNotYetStarted() {
    Account alice = sm.createAccount(100);
    BigInteger fund = ICX.multiply(BigInteger.valueOf(40));
    // crowdsale is not yet started
    assertThrows(AssertionError.class, () ->
            sm.transfer(alice, crowdsaleScore.getAddress(), fund));
  }

  @Test
  void fallback() {
    startCrowdsale();
    // fund 40 icx from Alice
    Account alice = sm.createAccount(100);
    BigInteger fund = ICX.multiply(BigInteger.valueOf(40));
    sm.transfer(alice, crowdsaleScore.getAddress(), fund);
    // verify
    verify(crowdsaleSpy).fallback();
    verify(crowdsaleSpy).FundTransfer(alice.getAddress(), fund, true);
    assertEquals(fund, Account.getAccount(crowdsaleScore.getAddress()).getBalance());
  }

  @Test
  void checkGoalReached() {
    startCrowdsale();
    // increase the block height
    sm.getBlock().increase(durationInBlocks.longValue());
    crowdsaleScore.invoke(owner, "checkGoalReached");
    // verify
    verify(crowdsaleSpy).CrowdsaleEnded();
    verify(crowdsaleSpy, never()).GoalReached(any(), any());
  }

  @Test
  void safeWithdrawal() {
    startCrowdsale();
    // fund 40 icx from Alice
    Account alice = sm.createAccount(100);
    sm.transfer(alice, crowdsaleScore.getAddress(), ICX.multiply(BigInteger.valueOf(40)));
    // fund 60 icx from Bob
    Account bob = sm.createAccount(100);
    sm.transfer(bob, crowdsaleScore.getAddress(), ICX.multiply(BigInteger.valueOf(60)));
    // make the goal reached
    sm.getBlock().increase(durationInBlocks.longValue());
    crowdsaleScore.invoke(owner, "checkGoalReached");
    // invoke safeWithdrawal
    crowdsaleScore.invoke(owner, "safeWithdrawal");
    // verify
    verify(crowdsaleSpy).GoalReached(owner.getAddress(), ICX.multiply(fundingGoalInICX));
    verify(crowdsaleSpy).FundTransfer(owner.getAddress(), ICX.multiply(fundingGoalInICX), false);
    assertEquals(ICX.multiply(fundingGoalInICX), Account.getAccount(owner.getAddress()).getBalance());
  }

  @Test
  void safeWithdrawal_refund() {
    startCrowdsale();
    // fund 40 icx from Alice
    Account alice = sm.createAccount(100);
    sm.transfer(alice, crowdsaleScore.getAddress(), ICX.multiply(BigInteger.valueOf(40)));
    // fund 50 icx from Bob
    Account bob = sm.createAccount(100);
    sm.transfer(bob, crowdsaleScore.getAddress(), ICX.multiply(BigInteger.valueOf(50)));
    // make the goal reached
    sm.getBlock().increase(durationInBlocks.longValue());
    crowdsaleScore.invoke(owner, "checkGoalReached");
    verify(crowdsaleSpy, never()).GoalReached(owner.getAddress(), ICX.multiply(fundingGoalInICX));

    // invoke safeWithdrawal from alice to refund
    crowdsaleScore.invoke(alice, "safeWithdrawal");
    verify(crowdsaleSpy).FundTransfer(alice.getAddress(), ICX.multiply(BigInteger.valueOf(40)), false);
    // invoke safeWithdrawal from bob to refund
    crowdsaleScore.invoke(bob, "safeWithdrawal");
    verify(crowdsaleSpy).FundTransfer(bob.getAddress(), ICX.multiply(BigInteger.valueOf(50)), false);
  }
}

```

