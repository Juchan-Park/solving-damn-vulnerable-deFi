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




