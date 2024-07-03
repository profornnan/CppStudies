# Item 39: private 상속은 심사숙고해서 구사하자

C++는 public 상속을 is-a 관계로 나타냄 → Item 32 참고

```c++
class Person { ... };
class Student: private Person { ... };  // private 상속

void eat(const Person& p);
void study(const Student& s);

Person p;
Student s;

eat(p);  // 문제 없음. p는 Person의 일종
eat(s);  // 에러! Student는 Person의 일종이 아님
```

private 상속은 is-a를 뜻하지 않음

클래스 사이의 상속 관계가 private이면 컴파일러는 일반적으로 파생 클래스 객체를 기본 클래스 객체로 변환하지 않음

기본 클래스로부터 물려받은 멤버는 파생 클래스에서 모조리 private 멤버가 됨

---

private 상속의 의미는 is-implemented-in-terms-of

B 클래스로부터 private 상속을 통해 D 클래스를 파생시킨 것은, B 클래스에서 쓸 수 있는 기능들 몇 개를 활용할 목적으로 한 행동이지, B 타입과 D 타입의 객체 사이에 어떤 개념적 관계가 있어서 한 행동이 아님

private 상속은 그 자체로 구현 기법 중 하나

private 상속의 의미는 '구현만 물려받을 수 있다. 인터페이스는 국물도 없다'라는 뜻

---

할 수 있으면 객체 합성을 사용하고, 꼭 해야 하면 private 상속 사용

꼭 해야 하는 때 → 비공개 멤버를 접근할 때 혹은 가상 함수를 재정의할 경우

공간 문제가 얽히면서 완전히 private 상속으로 기울 수 밖에 없는 만약의 경우도 있음

---

Widget 객체

멤버 함수 호출 횟수, 실행 시간이 지남에 따른 호출 비율 변화

실행 단계가 딱딱 구분되는 프로그램은 각 단계에 따라 보여주는 프로파일(profile) 양상도 다를 수 있음

프로파일(profile) : 프로그램이 실행되면서 호출되는 함수들의 순차적 리스트 및 각 함수의 실행 시간과 전체 시간에 대한 실행 시간 비율을 분석하는 작업 혹은 그 분석 결과. 프로파일러는 개발환경에 따라 다름

타이머 설치

```c++
class Timer {
public:
    explicit Timer(int tickFrequency);
    virtual void onTick() const;  // 일정 시간이 경과할 때마다 자동으로 호출됨
    ...
};
```

Timer 객체는 반복적으로 시간을 경과시킬 주기를 정해 줄 수 있고, 일정 시간이 경과할 때마다 가상 함수를 호출하도록 되어 있음

가상 함수를 재정의해서, Widget 객체의 현재 상태를 점검

Widget 클래스에서 Timer의 가상 함수를 재정의할 수 있어야 하므로, Widget 클래스는 Timer에서 상속을 받아야 함

public 상속은 맞지 않음

Widget이 Timer의 일종(is-a)은 아님

Widget 객체의 사용자는 Widget 객체를 통해 onTick 함수를 호출해선 안 됨 → 개념적으로 Widget 인터페이스의 일부로 볼 수 없기 때문

private 상속

```c++
class Widget: private Timer {
private:
    virtual void onTick() const;  // Widget 사용 자료 등을 수집
    ...
};
```

private 상속 → Timer의 public 멤버인 onTick 함수는 Widget에서 private 멤버가 되었음

---

객체 합성

Timer로부터 public 상속을 받은 클래스를 Widget 안에 private 중첩 클래스로 선언해 놓고, 이 클래스에서 onTick을 재정의한 다음, 이 타입의 객체 하나를 Widget 안에 데이터 멤버로서 넣는 것

```c++
class Widget {
private:
    class WidgetTimer: public Timer {
    public:
        virtual void onTick() const;
        ...
    };
    WidgetTimer timer;
    ...
};
```

하나의 설계 문제에 대한 접근 방법이 꼭 하나만 있는 것은 아님

여러 가지 방법을 실제로 고민하는 습관을 들이는 것이 좋음

---

현실적으로는 private 상속 대신에 public 상속에 객체 합성 조합이 더 자주 쓰임

Widget 클래스를 설계하는 데 있어서 파생은 가능하게 하되, 파생 클래스에서 onTick을 재정의할 수 없도록 설계 차원에서 막고 싶을 때 유용

Widget의 컴파일 의존성을 최소화하고 싶을 때 좋음

---

private 상속을 해야 할 경우

기본 클래스의 비공개 부분에 파생 클래스가 접근해야 한다거나 가상 함수를 한 개 이상 재정의해야 할 경우

클래스 사이의 개념적인 관계는 is-implemented-in-terms-of

공간 최적화가 얽힌 만약의 경우

데이터가 전혀 없는 클래스를 사용할 때가 아니면 볼 수도 없음

데이터가 없는 클래스란 비정적 데이터 멤버가 없는 클래스 → 가상 함수도 하나도 없어야 하고, 가상 기본 클래스도 없어야 함

이런 공백 클래스(empty class)는 개념적으로 차지하는 메모리 공간이 없는 게 맞음

하지만 C++에는 독립 구조(freestanding)의 객체는 반드시 크기가 0을 넘어야 한다는 금기사항 같은 것이 정해져 내려오고 있음

```c++
class Empty {};  // 정의된 데이터가 없으므로, 객체는 메모리를 사용하지 말아야 함

class HoldsAnInt {  // int를 저장할 공간만 필요해야 함
private:
    int x;
    Empty e;  // 메모리 요구가 없어야 함
};
```

sizeof(HoldsAnInt) > sizeof(int)가 됨

대부분의 컴파일러에서 sizeof(Empty)의 값은 1로 나옴

크기가 0인 독립 구조의 객체가 생기는 것을 금지하는 C++의 제약을 지키기 위해, 컴파일러는 이런 공백 객체에 char 한 개를 끼워넣는 식으로 처리하기 때문

하지만 바이트 정렬(item 50 참고)이 필요하다고 판단되면 컴파일러는 HoldsAnInt 등의 클래스에 바이트 패딩(padding) 과정을 추가할 수도 있어서, HoldsAnInt 객체의 크기는 두번째 int를 담을 수 있는 만큼으로 늘어남

이 C++의 제약은 파생 클래스 객체의 기본 클래스 부분에는 적용되지 않음 → 이때의 기본 클래스 부분은 독립구조 객체가 아니기 때문

Empty로부터 상속시키기

```c++
class HoldsAnInt: private Empty {
private:
    int x;
};
```

sizeof(HoldsAnInt) == sizeof(int)

공백 기본 클래스 최적화(empty base optimization: EBO)

EBO는 일반적으로 단일 상속하에서만 적용됨

---

실무적인 입장에서 공백 클래스는 진짜로 텅 빈 것은 아님

비정적 데이터 멤버는 안 갖고 있지만, typedef, enum, 정적 데이터 멤버, 비가상 함수를 갖는 경우가 많음

STL에는 방금 말한 성격의 멤버(대개 typedef 타입)를 포함하고 있는, 기술적으로 공백 처리된 클래스가 많이 있음

unary_function, binary_function → 사용자 정의 함수 객체를 만들 때 상속시킬 기본 클래스로 굉장히 자주 사용되는 클래스

요즘은 EBO의 구현이 보편화되어, 이런 상속은 아무리 자주 되더라도 파생 클래스의 크기를 증가시키는 일이 거의 없음

---

아무것도 없는 클래스를 사용하는 경우는 드물어서 EBO 하나만 갖고 private 상속이 정당화된 것인 양 생각하는 것은 무리에 가까움

대부분의 상속은 is-a 관계 → public 상속

is-implemented-in-terms-of 관계는 객체 합성과 private 상속으로 나타낼 수 있지만, 이해하기에는 객체 합성이 훨씬 나음

---

private 상속이 적법한 설계 전략일 가능성이 가장 높은 경우

is-a 관계로 이어질 것 같지 않은 두 클래스 사이에서 한쪽 클래스가 다른 쪽 클래스의 protected 멤버에 접근해야 하거나 다른 쪽 클래스의 가상 함수를 재정의해야 할 때

private 상속을 심사숙고해서 구사하자

모든 대안을 고민한 연후에, 주어진 상황에서 두 클래스 사이의 관계를 나타낼 가장 좋은 방법이 private 상속이라는 결론이 나면 쓰라는 뜻

---

**private 상속의 의미는 is-implemented-in-terms-of(...는 ...를 써서 구현됨)임. 대개 객체 합성과 비교해서 쓰이는 분야가 많지는 않지만, 파생 클래스 쪽에서 기본 클래스의 protected 멤버에 접근해야 할 경우 혹은 상속받은 가상 함수를 재정의해야 할 경우에는 private 상속이 나름대로 의미가 있음**

**객체 합성과 달리, private 상속은 공백 기본 클래스 최적화(EBO)를 활성화시킬 수 있음. 이 점은 객체 크기를 가지고 고민하는 라이브러리 개발자에게 꽤 매력적인 특징이 되기도 함**

