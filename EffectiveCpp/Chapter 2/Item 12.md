# Item 12: 객체의 모든 부분을 빠짐없이 복사하자

객체 복사 함수 → 복사 생성자, 복사 대입 연산자

컴파일러가 필요에 따라 만들어내기도 함

복사되는 객체가 갖고 있는 데이터를 빠짐없이 복사

---

고객 클래스

```c++
void logCall(const std::string& funcName);

class Date { ... };

class Customer {
public:
    ...
    Customer(const Customer& rhs);
    Customer& operator=(const Customer& rhs);
    ...

private:
    std::string name;
    Date lastTransaction;
};

Customer::Customer(const Customer& rhs): name(rhs.name) {
    logCall("Customer copy constructor");
}

Customer& Customer::operator=(const Customer& rhs) {
    logCall("Customer copy assignment operator");
    name = rhs.name;
    return *this;
}
```

lastTransaction 데이터 멤버 추가

부분 복사가 됨

이런 상황에 대해 알려주는 컴파일러 거의 없음

클래스에 데이터 멤버 추가 시, 추가한 데이터 멤버를 처리하도록 복사 함수 재작성, 생성자 갱신, 비표준형 operator= 함수 변경

---

파생 클래스에 대한 복사 함수 구현 시, 기본 클래스의 복사 함수 호출

```c++
class PriorityCustomer: public Customer {
public:
    ...
    PriorityCustomer(const PriorityCustomer& rhs);
    PriorityCustomer& operator=(const PriorityCustomer& rhs);
    ...

private:
    int priority;
};

PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs): Customer(rhs), priority(rhs.priority) {  // 기본 클래스 복사 생성자 호출
    logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs) {
    logCall("PriorityCustomer copy assignment operator");
    Customer::operator=(rhs);  // 기본 클래스 대입
    priority = rhs.priority;
    return *this;
}
```

---

모든 부분을 복사하자

1. 해당 클래스의 데이터 멤버 모두 복사
2. 이 클래스가 상속한 기본 클래스의 복사 함수 호출

---

복사 생성자, 복사 대입 연산자 → 한쪽에서 다른 쪽 호출 X

양쪽에서 겹치는 부분을 별도의 멤버 함수에 분리 후 이 함수를 호출해 코드 중복 제거

---

**객체 복사 함수는 주어진 객체의 모든 데이터 멤버 및 모든 기본 클래스 부분을 빠뜨리지 말고 복사해야 함**

**클래스의 복사 함수 두 개를 구현할 때, 한쪽을 이용해서 다른 쪽을 구현하려는 시도는 절대 하지 말자. 그 대신, 공통된 동작을 제3의 함수에 분리해 놓고 양쪽에서 이것을 호출하게 만들어서 해결하자.**

