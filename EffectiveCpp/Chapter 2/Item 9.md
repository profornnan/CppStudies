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

---

객체가 소멸될 때도 똑같음

파생 클래스의 소멸자가 호출되고 나면 파생 클래스만의 데이터 멤버는 정의되지 않은 값으로 가정됨

기본 클래스 소멸자 진입 시 기본 클래스 객체가 됨

---

위의 예제는 컴파일러가 경고 메시지를 주거나 프로그램 실행 전 문제가 드러날 수 있음

순수 가상 함수 → 함수의 정의가 존재하지 않으면 링크 단계 에러 발생

---

생성자에서 하는 똑같은 작업을 모아 공동의 초기화 코드 생성

비가상 초기화 함수 안에서 logTransaction을 호출한다고 가정

```c++
class Transaction {
public:
    Transaction() { init(); }  // 비가상 멤버 함수 호출
    virtual void logTransaction() const = 0;
    ...

private:
    void init() {
        ...
        logTransaction();  // 비가상 멤버 함수에서 가상 함수 호출
    }
};
```

컴파일도 잘 되고 링크도 잘 됨

대부분의 시스템은 순수 가상 함수가 호출될 때 프로그램을 바로 끝내버림

logTransaction가 순수 가상 함수가 아닐 경우 문제가 복잡해짐

생성자나 소멸자에서 가상 함수 호출 X

---

대처 방법

logTransaction을 Transaction 클래스의 비가상 멤버 함수로 변경

파생 클래스의 생성자가 필요한 로그 정보를 Transaction의 생성자로 넘기도록 함

```c++
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;
    ...
};

Transaction::Transaction(const std::string& logInfo) {
    ...
    logTransaction(logInfo);
}

class BuyTransaction: public Transaction {
public:
    BuyTransaction(parameters): Transaction(createLogString(parameters)) { ... }
    ...

private:
    static std::string createLogString(parameters);
};
```

필요한 초기화 정보를 파생 클래스 쪽에서 기본 클래스 생성자로 올려줌

createLogString는 기본 클래스 생성자 쪽으로 넘길 값을 생성하는 용도로 사용됨

기본 클래스의 멤버 초기화 리스트가 복잡한 경우 편리함

정적 멤버로 되어있음 → 미초기화된 데이터 멤버를 건드릴 위험 없음

**미초기화된 데이터 멤버는 정의되지 않은 상태에 있음**

---

**생성자 혹은 소멸자 안에서 가상 함수를 호출하지 말자. 가상 함수라고 해도, 지금 실행중인 생성자나 소멸자에 해당되는 클래스의 파생 클래스 쪽으로는 내려가지 않음**

