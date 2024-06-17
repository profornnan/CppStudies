# Item 32: public 상속 모형은 반드시 "is-a(...는 ...의 일종이다)"를 따르도록 만들자

C++로 객체 지향 프로그래밍을 하면서 꼭 잊지 말아야 할 중요한 규칙

public 상속은 "is-a(...는 ...의 일종이다)"를 의미함

클래스 D(Derived)를 클래스 B(Base)로부터 public 상속을 통해 파생시킴

D 타입으로 만들어진 모든 객체는 또한 B 타입의 객체이지만, 그 반대는 되지 않음

B는 D보다 더 일반적인 개념을 나타내며, D는 B보다 더 특수한 개념을 나타냄

B 타입의 객체가 쓰일 수 있는 곳에는 D 타입의 객체도 쓰일 수 있다고 단정(assert)하는 것

반면, D 타입이 필요한 부분에 B 타입의 객체를 쓰는 것을 불가능함

모든 D는 B의 일종이지만(D is a B), B는 D의 일종이 아니기 때문

---

C++ public 상속

```c++
class Person { ... };
class Student: public Person { ... };
```

사람은 학생보다 더 일반적인 개념

---

Person 타입(Person에 대한 포인터 혹은 Person에 대한 참조자도 가능)의 인자를 기대하는 함수는 Student 객체(Student에 대한 포인터 혹은 Student에 대한 참조자도 가능)도 받아들일 수 있음

```c++
void eat(const Person& p);
void study(const Student& s);

Person p;
Student s;

eat(p);  // 문제 없음
eat(s);  // 문제 없음. Student는 Person의 일종

study(s);  // 문제 없음
study(p);  // 에러! p는 Student가 아님
```

위와 같이 동작하려면 Student가 Person과 public 상속 관계를 가져야 함

private 상속은 의미 자체가 완전히 다름(Item 39 참고)

---

public 상속과 is-a 관계는 똑같은 뜻

판단을 잘못하는 경우도 있음

펭귄은 새의 일종

새는 날 수 있음

```c++
class Bird {
public:
    virtual void fly();
    ...
};

class Penguin: public Bird {
    ...
};
```

모든 종류의 새가 날 수 있다는 의미는 아님

날지 않는 새 종류 구분

```c++
class Bird {
    ...
};

class FlyingBird: public Bird {
public:
    virtual void fly();
    ...
};

class Penguin: public Bird {
    ...
};
```

어떤 소프트웨어 시스템의 경우엔 비행 능력이 있는 새와 없는 새를 구분하지 않아도 될 수 있음

소프트웨어에 이상적인 설계 같은 것은 없음

최고의 설계는, 제작하려는 소프트웨어 시스템이 기대하는 바에 따라 달라짐

---

펭귄의 fly 함수를 재정의해서 런타임 에러를 내도록 하는 것

"펭귄은 날 수 없다"가 아니라 "펭귄은 날 수 있다. 그러나 펭귄이 실제로 날려고 하면 에러가 난다"고 말하는 것

유효하지 않은 코드를 컴파일 단계에서 막아 주는 인터페이스가 좋은 인터페이스임 → Item 18 참고

펭귄의 무모한 비행을 컴파일 타임에 거부하는 설계가 그것을 런타임에 뒤늦게 알아채는 설계보다 훨씬 좋음

---

Square(정사각형) 클래스는 Rectangle(직사각형) 클래스로부터 상속받아야 할까?

```c++
class Rectangle {
public:
    virtual void setHeight(int newHeight);
    virtual void setWidth(int newWidth);
    
    virtual int height() const;
    virtual int width() const;
    ...
};

void makeBigger(Rectangle& r) {  // r의 넓이를 늘리는 함수
    int oldHeight = r.height();
    r.setWidth(r.width() + 10);
    assert(r.height() == oldHeight);  // r의 세로 길이가 변하지 않는다는 조건에 단정문을 걸어둠
}
```

위의 단정문이 실패할 일이 없다는 것은 확실함

public 상속을 써서 정사각형을 직사각형처럼 처리

```c++
class Square: public Rectangle { ... };
Square s;
...
assert(s.width() == s.height());
makeBigger(s);
assert(w.width() == s.height());
```

두 번째 단정문도 실패해서는 안 됨

단정문 조정이 필요함

- makeBigger 함수를 호출하기 전에, s의 세로 길이는 가로 길이와 같아야 함
- makeBigger 함수가 실행되는 중에, s의 가로 길이는 변하는데 세로 길이는 안 변해야 함
- makeBigger 함수에서 복귀한 후에, s의 세로 길이는 가로 길이와 같아야 함(참조에 의한 전달)

직사각형의 성질 중 어떤 것(가로 길이가 세로 길이에 상관없이 바뀔 수 있음)은 정사각형(가로와 세로 길이가 같아야 함)에 적용할 수 없음

public 상속은 기본 클래스 객체가 가진 모든 것들이 파생 클래스 객체에도 그대로 적용된다고 단정하는 상속

직사각형과 정사각형의 경우 이런 단정이 참이 될 수 없음

이 둘의 관계를 public 상속을 써서 표현하려고 하면 틀리는 것

문법적 하자가 없기 때문에 무사히 통과되지만 제대로 동작할 거라는 보장은 없음

---

클래스들 사이에 맺을 수 있는 관계로 is-a 관계만 있는 것은 아님

두 가지가 더 있음 → "has-a(...는 ...를 가짐)"와 "is-implemented-in-terms-of(...는 ...를 써서 구현됨)" → Item 38, Item 39 참고

is-a 이외의 나머지 두 관계를 is-a 관계로 모형화해서 설계가 이상하게 꼬이는 경우가 많이 있음

각각을 C++로 가장 잘 표현하는 방법 공부

---

**public 상속의 의미는 "is-a(...는 ...의 일종)"임. 기본 클래스에 적용되는 모든 것들이 파생 클래스에 그대로 적용되어야 함. 모든 파생 클래스 객체는 기본 클래스 객체의 일종이기 때문**

