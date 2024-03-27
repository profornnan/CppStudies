# Item 18: 인터페이스 설계는 제대로 쓰기엔 쉽게, 엉터리로 쓰기엔 어렵게 하자

함수, 클래스, 템플릿 → 인터페이스

인터페이스는 사용자가 우리의 코드와 만리장성을 쌓는 접선수단

인터페이스 사용 결과 코드가 사용자가 생각한대로 동작하지 않는다면 컴파일되지 않아야 함

어떤 코드가 컴파일된다면 사용자가 원하는 대로 동작해야 함

---

제대로 쓰기에 쉽고 엉터리로 쓰기에 어려운 인터페이스를 개발하려면 사용자가 저지를 만한 실수의 종류를 머리에 넣어두고 있어야 함

날짜 클래스 생성자 설계

```c++
class Date {
public:
    Date(int month, int day, int year);
    ...
};
```

매개변수 전달 순서가 잘못되거나 월과 일에 해당하는 숫자가 어이없는 숫자일 수 있음

새로운 타입을 들여와 인터페이스를 강화하면 상당수의 사용자 실수를 막을 수 있음

일, 월, 연을 구분하는 랩퍼(wrapper) 클래스

```c++
struct Day {
    explicit Day(int d): val(d) {}
    int val;
};

struct Month {
    explicit Month(int m): val(m) {}
    int val;
};

struct Year {
    explicit Year(int y): val(y) {}
    int val;
};

class Date {
public:
    Date(const Month& m, const Day& d, const Year& y);
    ...
};

Date d(Month(3), Day(30), Year(1995));
```

Day, Month, Year 온전한 클래스로 만들기 → Item 22 참고

적절한 타입 준비로 인터페이스 사용 에러 막을 수 있음

적절한 타입만 제대로 준비되어 있으면, 각 타입의 값에 제약을 가하더라도 괜찮은 경우가 생김

유효한 Month의 집합 미리 정의

타입 안전성

```c++
class Month {
public:
    static Month Jan() { return Month(1); }  // 유효한 Month 값 반환 함수들
    static Month Feb() { return Month(2); }
    ...
    static Month Dec() { return Month(12); }
    ...  // 다른 멤버 함수들
private:
    explicit Month(int m);  // Month 값이 새로 생성되지 않도록 명시호출 생성자가 private 멤버
    ...  // 월 표현을 위한 내부 데이터
};

Date d(Month::Mar(), Day(30), Year(1995));
```

특정한 월을 나타내는 데 객체를 쓰지 않고 함수 사용

비지역 정적 객체들의 초기화를 믿고 밀고 나가면 안 됨 → Item 4 참고

---

타입에 제약을 부여해 사용자 실수 방지

const 붙이기 → Item 3 참고

---

그렇게 하지 않을 번듯한 이유가 없다면 사용자 정의 타입은 기본제공 타입처럼 동작하게 만들자

아리송하면, int의 동작 원리대로 만들자

---

**일관성** 있는 인터페이스 제공을 위해 기본제공 타입과 쓸데없이 어긋나는 동작을 피하자

STL 컨테이너의 인터페이스는 전반적으로 일관성을 가지고 있음

---

사용자 쪽에서 뭔가를 외워야 제대로 쓸 수 있는 인터페이스는 잘못 쓰기 쉬움

Item 13 참고

```c++
Investment* createInvestment();
```

Investment 클래스 계통에 속해 있는 어떤 객체를 동적 할당하고 그 객체의 포인터를 반환하는 함수

자원 누출을 피하기 위해 createInvestment에서 얻어낸 포인터를 삭제해야 함

createInvestment의 반환 값을 스마트 포인터에 저장한 후에 해당 포인터의 삭제 작업을 스마트 포인터에 떠넘기는 방법 사용

스마트 포인터를 사용해야 한다는 사실도 잊어버리면?

팩토리 함수가 스마트 포인터를 반환하게 만들자

```c++
std::tr1::shared_ptr<Investment> createInvestment();
```

tr1::shared_ptr은 생성 시점에 자원 해제 함수(삭제자)를 직접 엮을 수 있기 때문에 tr1::shared_ptr을 반환하는 구조는 자원 해제 관련 실수를 사전 봉쇄할 수 있음 → Item 14 참고

---

createInvestment를 통해 얻은 Investment* 포인터를 직접 삭제하지 않게 하고 getRidOfInvestment 함수에 넘기게 하는 방법 → 자원 해제 메커니즘을 잘못 사용할 수 있음

getRidOfInvestment가 삭제자로 묶인 tr1::shared_ptr을 반환하도록 createInvestment 수정

tr1::shared_ptr의 첫 번째 인자는 스마트 포인터로 관리할 실제 포인터, 두 번째 인자는 참조 카운트가 0이 될 때 호출될 삭제자

```c++
std::tr1::shared_ptr<Investment> pInv(static_cast<Investment*>(0), getRidOfInvestment);
```

getRidOfInvestment를 삭제자로 갖는 널 shared_ptr 생성

static_cast → Item 27 참고

```c++
std::tr1::shared_ptr<Investment> createInvestment() {
    std::tr1::shared_ptr<Investment> retVal(static_cast<Investment*>(0), getRidOfInvestment);
    retVal = ...;  // retVal이 실제 객체를 가리키도록 만들기
    return retVal;
}
```

retVal로 관리할 실제 객체의 포인터를 결정하는 시점이 retVal을 생성하는 시점보다 앞설 수 있으면, retVal을 널로 초기화하고 나서 나중에 대입하는 방법보다 실제 객체의 포인터를 바로 retVal의 생성자에 넘겨버리는 게 더 좋음 → Item 26 참고

---

tr1::shared_ptr은 포인터별(per-pointer) 삭제자를 자동으로 사용해 '교차 DLL 문제'를 없애줌

객체 생성 시 사용한 동적 링크 라이브러리(dynamically linked library: DLL)와 다른 동적 링크 라이브러리의 delete를 사용해 객체를 삭제하는 경우 교차 DLL 문제 발생

new/delete 짝이 실행되는 DLL이 달라서 꼬이게 되면 대부분 런타임 에러 발생

tr1::shared_ptr의 기본 삭제자는 tr1::shared_ptr이 생성된 DLL과 동일한 DLL에서 delete를 사용하도록 만들어져 있음

---

tr1::shared_ptr을 사용하면 제대로 쓰기엔 쉽고 엉터리로 쓰기엔 어려운 인터페이스를 만드는 데 쉽게 다가갈 수 있음

tr1::shared_ptr을 구현한 제품 중 가장 흔히 쓰이는 것은 부스트 라이브러리 → Item 55 참고

이 클래스를 사용하면 원시 포인터보다 크고 느리며 내부 관리용 동적 메모리까지 추가로 사용됨

하지만 응용프로그램에서 런타임 비용이 눈에 띄게 늘어나는 경우는 별로 없음

사용자 실수가 눈에 띄게 줄어들게 됨

---

**좋은 인터페이스는 제대로 쓰기에 쉬우며 엉터리로 쓰기에 어려움**

**인터페이스의 올바른 사용을 이끄는 방법 → 인터페이스 사이의 일관성 잡아주기, 기본제공 타입과의 동작 호환성 유지하기**

**사용자의 실수를 방지하는 방법 → 새로운 타입 만들기, 타입에 대한 연산 제한하기, 객체의 값에 대해 제약 걸기, 자원 관리 작업을 사용자 책임으로 놓지 않기**

**tr1::shared_ptr은 사용자 정의 삭제자를 지원함 → 교차 DLL 문제를 막아 주며, 뮤텍스 등을 자동으로 잠금 해제하는데 쓸 수 있음**

