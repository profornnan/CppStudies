# Item 41: 템플릿 프로그래밍의 천릿길도 암시적 인터페이스와 컴파일 타임 다형성부터

객체 지향 프로그래밍의 세계

명시적 인터페이스(explicit interface), 런타임 다형성(runtime polymorphism)

```c++
class Widget {
public:
    Widget();
    virtual ~Widget();
    virtual std::size_t size() const;
    virtual void normalize();
    void swap(Widget& other);
    ...
};
```

```c++
void doProcessing(Widget& w) {
    if (w.size() > 10 && w != someNastyWidget) {
        Widget temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

doProcessing 함수 안에 있는 w

- w는 Widget 타입으로 선언되었기 때문에, w는 Widget 인터페이스를 지원해야 함. **명시적 인터페이스**. 소스 코드에 명시적으로 드러나는 인터페이스
- Widget의 멤버 함수 중 몇 개는 가상 함수이므로, 이 가상 함수에 대한 호출은 **런타임 다형성**에 의해 이루어짐. 특정한 함수에 대한 실제 호출은 w의 동적 타입(Item 37 참고)을 기반으로 프로그램 실행 중에 결정됨

---

템플릿과 일반화 프로그래밍의 세계

암시적 인터페이스(implicit interface), 컴파일 타임 다형성(compile-time polymorphism)

doProcessing 함수를 함수 템플릿으로 변경

```c++
template<typename T>
void doProcessing(T& w) {
    if (w.size() > 10 && w != someNastyWidget) {
        T temp(w);
        temp.normalize();
        temp.swap(w);
    }
}
```

doProcessing 템플릿 안에 있는 w

- w가 지원해야 하는 인터페이스는 이 템플릿 안에서 w에 대해 실행되는 연산이 결정함. 지금의 경우 size, normalize, swap 멤버 함수를 지원해야 하는 쪽은 w의 타입, 즉 T임. T는 복사 생성자, 부등 비교 연산도 지원해야 함. 이 템플릿이 제대로 컴파일되려면 몇 개의 표현식이 유효(valid)해야 하는데 이 표현식들은 바로 T가 지원해야 하는 **암시적 인터페이스**임
- w가 수반되는 함수 호출이 일어날 때, 이를테면 operator> 및 operator!= 함수가 호출될 때는 해당 호출을 성공시키기 위해 템플릿의 인스턴스화가 일어남. 이러한 인스턴스화가 일어나는 시점은 컴파일 도중임. 인스턴스화를 진행하는 함수 템플릿에 어떤 템플릿 매개변수가 들어가느냐에 따라 호출되는 함수가 달라지기 때문에, 이것을 가리켜 **컴파일 타임 다형성**이라고 함

---

명시적 인터페이스는 대개 함수 시그너처로 이루어짐

시그너처 : 함수의 이름, 매개변수 타입, 반환 타입 등을 통틀어 부르는 용어

Widget의 public 인터페이스는 생성자, 소멸자, size, normalize, swap 함수, 이들의 매개변수 타입, 반환 타입 및 각 함수의 상수성 여부로 이루어져 있음(컴파일러가 자동으로 만들어 놓은 복사 생성자와 복사 대입 연산자도 포함됨)

typedef 타입이 있을 경우에는 이것도 포함될 수 있음

데이터 멤버의 경우 시그너처에 들어가지 않음

---

암시적 인터페이스는 함수 시그너처에 기반하고 있지 않다는 것이 가장 큰 차이점

암시적 인터페이스를 이루는 요소는 유효 표현식(expression)임

doProcessing 템플릿의 시작부분에 있는 조건문

T(w의 타입)에서 제공될 암시적 인터페이스에는 다음과 같은 제약이 걸릴 것임

- 정수 계열의 값을 반환하고 이름이 size인 함수를 지원해야 함
- T 타입의 객체 둘을 비교하는 operator!= 함수를 지원해야 함

실제로는 연산자 오버로딩의 가능성이 있기 때문에 T는 위의 두 가지 제약 중 어떤 것도 만족시킬 필요가 없음

첫 번째 제약

T가 size 멤버 함수를 지원해야 하는 것은 맞지만 수치 타입을 반환할 필요까지는 없음. operator>의 정의에 필요한 타입도 반환할 필요가 없음. size 멤버 함수가 해야 하는 일은 어떤 X 타입의 객체와 int가 함께 호출될 수 있는 operator>가 성립될 수 있도록, X 타입의 객체만 반환하면 됨

operator> 함수는 반드시 X 타입의 매개변수를 받아들일 이유가 없음. 이 함수가 Y 타입의 매개변수를 받도록 정의되어 있고 X 타입에서 Y 타입으로 암시적인 변환이 가능하면 됨

두 번째 제약도 필수 요구사항이 되지 않음

operator!= 함수가 X 타입의 객체 하나와 Y 타입의 객체 하나를 받아들이면 됨

T가 X로 변환될 수 있으며 someNastyWidget의 타입이 Y로 변환되는 것이 가능하기만 하면 operator!= 함수의 호출은 유효 호출로 간주될 것임

---

암시적 인터페이스는 유효 표현식의 집합으로 구성되어 있음

표현식 자체만 보면 복잡해 보일 수도 있지만, 표현식에 걸리는 제약은 일반적으로 지극히 평이함

---

클래스에서 제공하는 명시적 인터페이스와 호환되지 않는 방법으로 그 클래스의 객체를 쓸 수 없듯이, 어떤 템플릿 안에서 어떤 객체를 쓰려고 할 때 그 템플릿에서 요구하는 암시적 인터페이스를 그 객체가 지원하지 않으면 사용이 불가능함

---

**클래스 및 템플릿은 모두 인터페이스와 다형성을 지원함**

**클래스의 경우, 인터페이스는 명시적이며 함수의 시그너처를 중심으로 구성되어 있음. 다형성은 프로그램 실행 중에 가상 함수를 통해 나타남**

**템플릿 매개변수의 경우, 인터페이스는 암시적이며 유효 표현식에 기반을 두어 구성됨. 다형성은 컴파일 중에 템플릿 인스턴스화와 함수 오버로딩 모호성 해결을 통해 나타남**

