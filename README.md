# solving-damn-vulnerable-deFi
Damn Vulnerable DeFi를 풀면서 겪은 시행착오들

## 1. Unstoppable
---
### 첫 번째 시도
ReceiverUnstoppable 컨트랙트를 거치지 않고 직접 `flashloan()` 함수를 호출한다면 pool에 있는 토큰을 모두 가져올 수 있지 않을까?
```js
Error: Transaction reverted: function call to a non-contract account
```
위 에러가 발생하면서 실패.

<br/>

### 두 번째 시도
단순히 flashLoan 함수가 제대로 안 돌아가게 하는 것에 초점을 맞춘다면?
```js
assert(poolBalance == balanceBefore);
```
`depositTokens()`을 통해서만 총량이 늘어나는 `poolBalance`와 balanceBefore의 값이 달라지도록 하면 flashLoan 함수는 실행되지 않을 것이다.

```js
await this.token.transfer(this.pool.address, INITIAL_ATTACKER_TOKEN_BALANCE);
```
`depositTokens()`을 호출하지 않고 직접 컨트랙트에 토큰을 보냄으로써 해결 완료.

<br/>

## 2. Naive receiver
---
### 첫 번째 시도
`receiveEther`를 여러번 호출해서 `pool.sendValue(amountToBeRepaid);`를 여러번 실행하면 되지 않을까? `sendValue`는 gas limit도 없어서 컨트랙트의 자금이 없어질 때까지 반복해서 실행될 것이다. 그런데 `receiveEther()`는 pool 만 호출할 수 있다. 따라서 이 방법은 패스.

<br/>

### 두 번째 시도
internal로 선언된 `_executeActionDuringFlashLoan()` 함수를 외부에서 상속받아서 새로 정의하는 것은 어떨까? 하지만 현재 컨트랙트에 직접적으로 호출할 수는 없다. 이것도 패스.

<br/>

### 세 번째 시도
현재 `flashLoan()`을 호출할 때 `borrowAmount`의 값은 컨트랙트 잔고 이하이기만 하면 된다. 다시 말해서, 0도 가능하다는 얘기. 그렇다면 10번 호출해서 user의 자금을 수수료로 다 태우게 하면 되지 않을까?
```js
  it("Exploit", async function () {
    /** CODE YOUR EXPLOIT HERE */
    for (let i = 0; i < 10; i++) {
      await this.pool.flashLoan(this.receiver.address, 0);
    }
  });
```
ethers 문법에서 시간이 조금 걸렸지만 해결 완료 :)


## 3. Truster

## 4. Side entrance

### 첫 번째 시도
`flashLoan`을 다시 실행 하는 re-entrancy 공격으로 접근했지만 `out of gas error`가 나면서 revert 됨

<br>

### 두 번째 시도
try/catch 문을 사용해서 해결하려 했지만 실패.

### 모범 답안
먼저 `flashLoan`을 호출하고 execute 함수를 만들어서 다시 풀에 deposit한다. 이후 `flashLoan`이 끝나면 attack 함수 내에서 자금을 인출하고, receive 함수를 이용해 attacker에게 자금을 송금한다. 매우 깔끔한 방법.

-> 한 번에 공격을 끝내는 방식으로 접근. 앞으로는 컨트랙트 내의 다른 함수를 활용하는 공격 방법도 고민해보자. 
```IFlashLoanEtherReceiver(msg.sender).execute{value: amount}();``` 
이 부분에서 msg.sender가 IFlashLoanEtherReceiver 타입으로 변하면서 execute를 실행할 수 있다는 것에 초점을 맞췄어야 했다. 단순히 함수를 하나 더 실행해서 이더를 옮기는 것으로 해석해서 잘못된 방향으로 접근했다. 의미 없는 라인은 없으니 한 줄을 읽을 때 제대로 해석하자.




