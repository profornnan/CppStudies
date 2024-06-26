# Item 37: 어떤 함수에 대해서도 상속받은 기본 매개변수 값은 절대로 재정의하지 말자

비가상 함수는 언제라도 재정의해서는 안 되는 함수 → Item 36 참고

기본 매개변수 값을 가진 가상 함수를 상속하는 경우

가상 함수는 동적으로 바인딩되지만, 기본 매개변수 값은 정적으로 바인딩됨

정적 바인딩은 선행 바인딩(early binding), 동적 바인딩은 지연 바인딩(late binding)

---

객체의 정적 타입(static type)은 프로그램 소스 안에 놓은 선언문을 통해 그 객체가 갖는 타입

```c++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    virtual void draw(ShapeColor color = Red) const = 0;
    ...
};

class Rectangle: public Shape {
public:
    virtual void draw(ShapeColor color = Green) const;  // 기본 매개변수 값이 달라짐
    ...
};

class Circle: public Shape {
public:
    virtual void draw(ShapeColor color) const;
    ...
};
```

```c++
Shape *ps;  // 정적 타입 = Shape*
Shape *pc = new Circle;  // 정적 타입 = Shape*
Shape *pr = new Rectangle;  // 정적 타입 = Shape*
```

---

객체의 동적 타입(dynamic type)은 현재 그 객체가 진짜로 무엇이냐에 따라 결정되는 타입

'이 객체가 어떻게 동작할 것이냐'를 가리키는 타입

pc의 동적 타입은 Circle\*, pr의 동적 타입은 Rectangle\*

동적 타입은 프로그램 실행 도중 바뀔 수 있음

대개 대입문을 통해 바뀜

---

가상 함수는 동적으로 바인딩됨 → 호출이 일어난 객체의 동적 타입에 따라 어떤 함수가 호출될지가 결정된다는 뜻

```c++
pc->draw(Shape::Red);  // Circle::draw(Shape::Red) 호출
pr->draw(Shape::Red);  // Rectangle::draw(Shape::Red) 호출
```

---

가상 함수는 동적으로 바인딩되어 있지만 기본 매개변수는 정적으로 바인딩되어 있음

파생 클래스에 정의된 가상 함수를 호출하면서 기본 클래스에 정의된 기본 매개변수 값을 사용해 버릴 수 있음

``` c++
pr->draw();  // Rectangle::draw(Shape::Red) 호출
```

pr의 동적 타입은 Rectangle\* → 호출되는 가상 함수는 Rectangle의 것

Rectangle::draw 함수에서는 기본 매개변수 값이 Green으로 되어 있음

하지만 pr의 정적 타입은 Shape\*이기 때문에, 기본 매개변수 값을 Shape 클래스에서 가져옴

Shape 및 Rectangle 클래스 양쪽에서 선언된 것이 한데 섞이는 함수 호출이 이루어짐

---

draw 함수가 가상 함수이고, 기본 매개변수 값들 중 하나가 파생 클래스에서 재정의되면 문제가 생김

---

런타임 효율

만약에 함수의 기본 매개변수가 동적으로 바인딩된다면, 프로그램 실행 중에 가상 함수의 기본 매개변수 값을 결정할 방법을 컴파일러 쪽에서 마련해야 함 → 컴파일 과정에서 결정하는 현재의 메커니즘보다는 느리고 복잡함

---

기본 클래스 및 파생 클래스의 기본 매개변수 값을 똑같이 제공하려고 하면 코드 중복에 의존성까지 걸림

Item 35의 비가상 인터페이스(non-virtual interface) 관용구(NVI 관용구) 사용

파생 클래스에서 재정의할 수 있는 가상 함수를 private 멤버로 두고, 이 가상 함수를 호출하는 public 비가상 함수를 기본 클래스에 만들어 두는 것

비가상 함수가 기본 매개변수를 지정

비가상 함수의 내부에서는 진짜 일을 맡은 가상 함수 호출

```c++
class Shape {
public:
    enum ShapeColor { Red, Green, Blue };
    void draw(ShapeColor color = Red) const { doDraw(color); }
    ...
private:
    virtual void doDraw(ShapeColor color) const = 0;
};

class Rectangle: public Shape {
public:
    ...
private:
    virtual void doDraw(ShapeColor color) const;
    ...
};
```

---

**상속받은 기본 매개변수 값은 절대로 재정의해서는 안 됨. 기본 매개변수 값은 정적으로 바인딩되는 반면, 가상 함수는 동적으로 바인딩되기 때문**

