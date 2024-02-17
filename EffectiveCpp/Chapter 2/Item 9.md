# Item 9: 객체 생성 및 소멸 과정 중에는 절대로 가상 함수를 호출하지 말자

객체 생성 및 소멸 과정 중에는 가상 함수를 호출하면 절대로 안 됨

호출 결과가 원하는 대로 돌아가지 않을 것임

---

주식 거래를 본떠 만든 클래스 계통 구조

```c++
class Transaction {  // 모든 거래에 대한 기본 클래스
public:
    Transaction();
    virtual void logTransaction() const = 0;  // 타입에 따라 달라지는 로그 기록
    ...
};

Transaction::Transaction() {  // 기본 클래스 생성자 구현
    ...
    logTransaction();
}

class BuyTransaction: public Transaction {  // Transaction의 파생 클래스
public:
    virtual void logTransaction() const;
    ...
};

class SellTransaction: public Transaction {  // Transaction의 파생 클래스
public:
    virtual void logTransaction() const;
    ...
};
```

```c++
BuyTransaction b;
```

파생 클래스 객체 생성 시 기본 클래스 부분이 먼저 호출됨

Transaction 생성자 마지막에 logTransaction 호출 → BuyTransaction의 것이 아니라 Transaction의 것이 호출됨

기본 클래스의 생성자가 호출될 동안에는, 가상 함수는 절대로 파생 클래스 쪽으로 내려가지 않음

그 대신, 객체 자신이 기본 클래스 타입인 것처럼 동작

기본 클래스 생성자가 돌아가고 있을 시점에 파생 클래스 데이터 멤버는 아직 초기화된 상태가 아님

**파생 클래스 객체의 기본 클래스 부분이 생성되는 동안은, 그 객체의 타입은 기본 클래스**

