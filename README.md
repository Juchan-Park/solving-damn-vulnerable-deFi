# solving-damn-vulnerable-deFi
Damn Vulnerable DeFi를 풀면서 겪은 시행착오들

## 1. Unstoppable
---
### 첫 번째 시도
ReceiverUnstoppable 컨트랙트를 거치지 않고 직접 `flashloan()` 함수를 호출한다면 pool에 있는 토큰을 모두 가져올 수 있지 않을까?
```
Error: Transaction reverted: function call to a non-contract account
```
위 에러가 발생하면서 실패.

<br/>

### 두 번째 시도
단순히 flashLoan 함수가 제대로 안 돌아가게 하는 것에 초점을 맞춘다면?
```
assert(poolBalance == balanceBefore);
```
`depositTokens()`을 통해서만 총량이 늘어나는 `poolBalance`와 balanceBefore의 값이 달라지도록 하면 flashLoan 함수는 실행되지 않을 것이다.

```
await this.token.transfer(this.pool.address, INITIAL_ATTACKER_TOKEN_BALANCE);
```
`depositTokens()`을 호출하지 않고 직접 컨트랙트에 토큰을 보냄으로써 해결 완료.
