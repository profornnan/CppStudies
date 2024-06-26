# Item 27: 캐스팅은 절약, 또 절약! 잊지 말자

C++의 동작 규칙은 '어떤 일이 있어도 타입 에러가 생기지 않도록 보장한다.'는 철학을 바탕으로 설계되어 있음

이론적으로 C++ 프로그램은 일단 컴파일만 깔끔하게 끝나면 그 이후엔 어떤 객체에 대해서도 불안전한 연산이나 말도 안 되는 연산을 수행하려 들지 않는다는 것

---

캐스트(cast)

C++에서 캐스팅은 정말로 조심해서 써야 하는 기능

---

**구형 스타일의 캐스트**

C 스타일 캐스트

```
(T)표현식
```

함수 방식 캐스트

```
T(표현식)
```

표현식 부분을 T 타입으로 캐스팅

---

**C++ 스타일의 캐스트**

```
const_cast<T>(표현식)
dynamic_cast<T>(표현식)
reinterpret_cast<T>(표현식)
static_cast<T>(표현식)
```

- const_cast
  - 객체의 상수성(constness)을 없애는 용도로 사용
  - 이런 기능을 가진 C++ 스타일의 캐스트는 이것밖에 없음
- dynamic_cast
  - 안전한 다운캐스팅(safe downcasting)을 할 때 사용하는 연산자
  - 주어진 객체가 어떤 클래스 상속 계통에 속한 특정 타입인지 아닌지를 결정하는 작업에 쓰임
  - 구형 스타일의 캐스트 문법으로는 흉내조차 낼 수 없는 유일한 캐스트
  - 런타임 비용이 높음
- reinterpret_cast
  - 포인터를 int로 바꾸는 등의 하부 수준 캐스팅을 위해 만들어진 연산자
  - 적용 결과는 구현환경에 의존적
  - 이런 캐스트는 하부 수준 코드 외에는 거의 없어야 함
- static_cast
  - 암시적 변환(비상수 객체를 상수 객체로 바꾸거나, int를 double로 바꾸는 등의 변환)을 강제로 진행할 때 사용
  - 타입 변환을 거꾸로 수행하는 용도(void*를 일반 타입의 포인터로 바꾸거나, 기본 클래스의 포인터를 파생 클래스의 포인터로 바꾸는 등)로도 쓰임

---

C++ 스타일의 캐스트를 쓰는 것이 바람직함

코드를 읽을 때 알아보기 쉬움

캐스트를 사용한 목적을 더 좁혀서 지정하기 때문에 컴파일러 쪽에서 사용 에러를 진단할 수 있음

---

타입 변환이 있으면 이로 말미암아 런타임에 실행되는 코드가 만들어지는 경우가 적지 않음

```c++
int x, y;
double d = static_cast<double>(x)/y;
```

int 타입의 x를 double 타입으로 캐스팅한 부분에서 코드가 만들어짐

---

```c++
class Base { ... };
class Derived: public Base { ... };
Derived d;
Base *pb = &d;  // Derived* → Base*의 암시적 형변환이 이루어짐
```

파생 클래스 객체에 대한 기본 클래스 포인터를 만드는 코드

두 포인터의 값이 같지 않을 때도 가끔 있음

포인터의 변위(offset)를 Derived* 포인터에 적용하여 실제 Base* 포인터 값을 구하는 동작이 런타임에 이루어짐

C++에서는 다중 상속이 사용되면 이런 현상이 항상 생기지만, 단일 상속인데도 이렇게 되는 경우가 있음

C++를 쓸 때는 데이터가 어떤 식으로 메모리에 박혀 있을 거라는 섣부른 가정을 피해야 하며, 이런 가정에 기반한 캐스팅은 미정의 동작을 낳을 수 있음

객체의 메모리 배치구조를 결정하는 방법과 객체의 주소를 계산하는 방법은 컴파일러마다 천차만별임

---

캐스팅이 들어가면 보기엔 맞는 것 같지만 실제로는 틀린 코드를 쓰고도 모르는 경우가 많아짐

가상 함수를 파생 클래스에서 재정의해서 구현할 때 기본 클래스의 버전을 가장 먼저 호출

Window 기본 클래스

SpecialWindow 파생 클래스

onResize 가상 함수를 모두 정의하고 있음

SpecialWindow의 onResize에서 Window의 onResize 호출

보기엔 맞는 것 같지만 실제로는 틀린 코드

```c++
class Window {
public:
    virtual void onResize() { ... };
    ...
};

class SpecialWindow: public Window {
public:
    virtual void onResize() {
        static_cast<Window>(*this).onResize();
        ...
    }
    ...
};
```

캐스팅이 일어나면서 *this의 기본 클래스 부분에 대한 사본이 임시적으로 만들어짐

임시 객체의 onResize 호출

수정이 반영되는 쪽은 현재 객체의 사본

---

이 문제를 풀려면 캐스팅을 빼버려야 함

그냥 현재 객체에 대고 onResize의 기본 클래스 버전을 호출

```c++
class SpecialWindow: public Window {
public:
    virtual void onResize() {
        Window::onResize();
        ...
    }
    ...
};
```

캐스트 연산자가 입맛 당기는 상황이라면 뭔가 꼬여가는 징조

---

dynamic_cast

클래스 이름에 대한 문자열 비교 연산에 기반을 둠

깊이가 4인 단일 상속 계통에 속한 어떤 객체에 대해 dynamic_cast 적용 시, 이름 비교를 위해 strcmp가 최대 네 번까지 불릴 수 있음

사용 시 주의해야 함

---

파생 클래스의 함수를 호출하고 싶은데, 그 객체를 조작할 수 있는 수단으로 기본 클래스의 포인터밖에 없을 경우

첫 번째 방법은, 파생 클래스 객체에 대한 포인터를 컨테이너에 담아둠으로써 각 객체를 기본 클래스 인터페이스를 통해 조작할 필요를 아예 없애 버리는 것. Window에서 파생된 다른 타입의 포인터를 담으려면 타입 안전성을 갖춘 컨테이너 여러 개가 필요함

두 번째 방법은, 원하는 조작을 가상 함수 집합으로 정리해서 기본 클래스에 넣는 것. ex) blink 함수가 SpecialWindow에서만 가능하지만, 아무것도 안 하는 기본 blink를 구현해서 가상 함수로 제공하는 것

상당히 많은 상황에서 dynamic_cast 대신 사용 가능

---

폭포식(cascading) dynamic_cast 설계는 피해야 함

---

정말 잘 작성된 C++ 코드는 캐스팅을 거의 쓰지 않음

캐스팅은 최대한 격리시키자

캐스팅을 해야 하는 코드를 내부 함수 속에 몰아 놓고, 이 함수를 호출하는 외부에서 알 수 없도록 인터페이스로 막기

---

**캐스팅은 되도록 피하자. 특히 수행 성능에 민감한 dynamic_cast는 몇 번이고 다시 생각. 설계 중 캐스팅 필요 시, 캐스팅을 쓰지 않는 다른 방법 시도**

**캐스팅이 필요하다면 함수 안에 숨기자. 사용자는 캐스팅을 넣지 않고 함수 호출 가능**

**구형 스타일 캐스트보다는 C++ 스타일의 캐스트를 선호하자. 발견하기도 쉽고, 설계자가 어떤 역할을 의도했는지 더 자세하게 드러남**

