# Item 42: typename의 두 가지 의미를 제대로 파악하자

템플릿의 타입 매개변수를 선언할 때는 class와 typename의 뜻이 완전히 똑같음

```c++
template<class T> class Widget;
template<typename T> class Widget;
```

---

typename을 쓰지 않으면 안 되는 때가 있음

함수 템플릿이 하나 있다고 가정

STL과 호환되는 컨테이너를 받아들이도록 만들어졌고, 이 컨테이너에 담기는 객체는 int에 대입할 수 있음

컨테이너에 담긴 원소들 중 두 번째 값 출력

```c++
template<typename C>
void print2nd(const C& container) {
    if (container.size() >= 2) {
        C::const_iterator iter(container.begin());
        ++iter;
        int value = *iter;
        std::cout << value;
    }
}
```

지역 변수 두 개 → iter, value

iter의 타입은 C::const_iterator인데, 템플릿 매개변수인 C에 따라 달라지는 타입임

템플릿 내의 이름 중에 이렇게 템플릿 매개변수에 종속된 것을 가리켜 **의존 이름(dependent name)**이라고 함

**중첩 의존 이름(nested dependent name)** : 의존 이름이 어떤 클래스 안에 중첩되어 있는 경우

C::const_iterator는 중첩 의존 타입 이름

value는 int 타입

int는 템플릿 매개변수가 어떻든 상관없는 타입 이름임 → **비의존 이름(non-dependent name)**

---

코드 안에 중첩 의존 이름이 있으면 컴파일러가 구문분석을 할 때 애로사항이 생김

```c++
template<typename C>
void print2nd(const C& container) {
    C::const_iterator * x;
    ...
}
```

C::const_iterator에 대한 포인터인 지역 변수로서 x를 선언하고 있는 것 같음

그런데 C::const_iterator가 타입이 아니라면?

const_iterator라는 이름을 가진 정적 데이터 멤버가 C에 들어 있다고도 볼 수 있음

그리고 x가 다른 전역 변수의 이름이라면 위의 코드는 C::const_iterator와 x를 피연산자로 한 곱샘 연산임

C의 정체가 무엇인지 다른 곳에서 알려 주지 않으면, C::const_iterator가 진짜 타입인지 아닌지를 알아낼 방법은 없음

print2nd 함수 템플릿이 구문분석기에 의해 처리되는 순간에도 C의 정체는 저절로 밝혀지지 않음

이때 C++는 모호성을 해결하기 위해 어떤 규칙을 하나 사용함 → 중첩 의존 이름은 기본적으로 타입이 아닌 것으로 해석됨

```c++
template<typename C>
void print2nd(const C& container) {
    if (container.size() >= 2) {
        C::const_iterator iter(container.begin());  // 이 이름은 타입이 아닌 것으로 가정함
        ...
    }
}
```

iter의 선언이 선언으로서 의미가 있으려면 C::const_iterator가 반드시 타입이어야 하는데, C++는 제멋대로 타입이 아닌 것으로 가정

C::const_iterator 앞에다가 typename 키워드를 붙여서 타입이라고 말해 주기

```c++
template<typename C>
void print2nd(const C& container) {
    if (container.size() >= 2) {
        typename C::const_iterator iter(container.begin());
        ...
    }
}
```

템플릿 안에서 중첩 의존 이름을 참조할 경우에는, 그 이름 앞에 typename 키워드를 붙이자

---

typename 키워드는 중첩 의존 이름만 식별하는 데 써야 함

```c++
template<typename C>
void f(const C& container, typename C::iterator iter);
```

C는 중첩 의존 타입 이름이 아니기 때문에 typename을 붙이면 안 됨

C::iterator는 중첩 의존 이름이기 때문에 typename이 반드시 붙어야 함

---

typename은 중첩 의존 타입 이름 앞에 붙여 주어야 한다는 규칙에 예외가 하나 있음

중첩 의존 타입 이름이 기본 클래스의 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로서 있을 경우에는 typename을 붙여 주면 안 됨

```c++
template<typename T>
class Derived: public Base<T>::Nested {  // 상속되는 기본 클래스 리스트: typename 쓰면 안 됨
public:
    explicit Derived(int x): Base<T>::Nested(x) {  // 멤버 초기화 리스트에 있는 기본 클래스 식별자: typename 쓰면 안 됨
        typename Base<T>::Nested temp;  // 중첩 의존 타입 이름이며 기본 클래스 리스트에도 없고 멤버 초기화 리스트의 기본 클래스 식별자도 아님: typename 필요
        ...
    }
    ...
};
```

---

어떤 함수 템플릿을 만들고 있는데, 매개변수로 넘어온 반복자가 가리키는 객체의 사본을 temp라는 이름의 지역변수로 만들어 놓고 싶은 경우

```c++
template<typename IterT>
void workWithIterator(IterT iter) {
    typename std::iterator_traits<IterT>::value_type temp(*iter);
    ...
}
```

`std::iterator_traits<IterT>::value_type` → C++ 표준의 특성정보(traits) 클래스(Item 47 참고) 사용

IterT 타입의 객체로 가리키는 대상의 타입이란 뜻

중첩 의존 타입이므로, 이름 앞에 typename을 써 주어야 함

typedef 이름을 만들고 싶은 경우

특성정보 클래스에 속한 value_type 등의 멤버 이름(Item 47 참고)에 대해 typedef 이름을 만들 때는 그 멤버 이름과 똑같이 짓는 것이 관례

```c++
template<typename IterT>
void workWithIterator(IterT iter) {
    typedef typename std::iterator_traits<IterT>::value_type value_type;
    value_type temp(*iter);
    ...
}
```

---

typename에 관한 규칙을 얼마나 강조하느냐는 컴파일러마다 조금씩 차이가 있음

---

**템플릿 매개변수를 선언할 때, class 및 typename은 서로 바꾸어 써도 무방함**

**중첩 의존 타입 이름을 식별하는 용도에는 반드시 typename을 사용함. 단, 중첩 의존 이름이 기본 클래스 리스트에 있거나 멤버 초기화 리스트 내의 기본 클래스 식별자로 있는 경우에는 예외임**

