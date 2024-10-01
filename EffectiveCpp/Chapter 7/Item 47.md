# Item 47: 타입에 대한 정보가 필요하다면 특성정보 클래스를 사용하자

STL은 기본적으로 컨테이너(container) 및 반복자(iterator), 알고리즘(algorithm)의 템플릿으로 구성되어 있지만, 이 외에 유틸리티(utility)라고 불리는 템플릿도 몇 개 들어 있음

advance 템플릿 : 지정된 반복자를 지정된 거리만큼 이동

+= 연산을 지원하는 반복자는 임의 접근 반복자밖에 없음

---

STL 반복자는 각 반복자가 지원하는 연산에 따라 다섯 개의 범주로 나뉨

### 입력 반복자(input iterator)

전진만 가능, 한 번에 한 칸씩만 이동, 자신이 가리키는 위치에서 읽기만 가능함, 읽을 수 있는 횟수가 한 번 뿐임

입력 파일에 대한 읽기 전용 파일 포인터를 본떠서 만듦

C++ 표준 라이브러리의 istream_iterator가 대표적임

### 출력 반복자(output iterator)

입력 반복자와 비슷하지만 출력용인 점만 다름

전진만 가능, 한 번에 한 칸씩만 이동, 자신이 가리키는 위치에서 쓰기만 가능함, 쓸 수 있는 횟수가 한 번 뿐임

출력 파일에 대한 쓰기 전용 파일 포인터를 본떠서 만듦

ostream_iterator가 대표적임

입력 반복자와 출력 반복자는 STL의 5대 반복자 범주 가운데 기능적으로 가장 처짐

단일 패스(one-pass) 알고리즘에만 제대로 쓸 수 있음

### 순방향 반복자(forward iterator)

입력 반복자와 출력 반복자가 하는 일은 기본적으로 다 할 수 있음

자신이 가리키고 있는 위치에서 읽기와 쓰기를 동시에 할 수 있으며, 여러 번이 가능함

다중 패스(multi-pass) 알고리즘에 사용 가능

STL은 원칙적으로 단일 연결 리스트를 제공하지 않지만 몇몇 라이브러리를 보면 제공하는 것들이 있는데(대개 이름이 slist), 이 컨테이너에 쓰는 반복자가 순방향 반복자

### 양방향 반복자(bidirectional iterator)

순방향 반복자에 뒤로 갈 수 있는 기능을 추가한 것

STL의 list에 쓰는 반복자

set, multiset, map, multimap 등의 컨테이너에도 사용

### 임의 접근 반복자(random access iterator)

양방향 반복자에 반복자 산술 연산 수행 기능을 추가한 것

반복자를 임의의 거리만큼 앞뒤로 이동시키는 일을 상수 시간 안에 할 수 있음

포인터 산술 연산과 비슷함

기본제공 포인터를 본떠서 임의 접근 반복자를 만듦

vector, deque, string에 사용하는 반복자

---

C++ 표준 라이브러리에는 다섯 개의 반복자 범주 각각을 식별하는 데 쓰이는 태그 구조체가 정의되어 있음

```c++
struct input_iterator_tag {};
struct output_iterator_tag {};
struct forward_iterator_tag: public input_iterator_tag {};
struct bidirectional_iterator_tag: public forward_iterator_tag {};
struct random_access_iterator_tag: public bidirectional_iterator_tag {};
```

is-a 상속 관계

---

advance 구현

```c++
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
    if (iter가 임의 접근 반복자이다) {
        iter += d;
    }
    else {
        if (d >= 0) { while (d--) ++iter; }
        else { while (d++) --iter; }
    }
}
```

임의 접근 반복자 → 반복자 산술 연산

다른 종류의 반복자 → ++ 혹은 -- 연산의 반복 호출

iter 부분이 임의 접근 반복자인지를 판단할 수 있는 수단이 있어야 함

어떤 타입에 대한 정보를 얻어낼 필요가 있음

특성정보(traits) : 컴파일 도중에 어떤 주어진 타입의 정보를 얻을 수 있게 하는 객체를 지칭하는 개념

특성정보는 C++에 미리 정의된 문법구조가 아니며, 키워드도 아님

C++ 프로그래머들이 따르는 구현 기법이며, 관례임

특성정보는 기본제공 타입과 사용자 정의 타입에서 모두 돌아가야 함 → 특성정보 기법을 포인터 등의 기본제공 타입에 적용할 수 있어야 함

어떤 타입의 특성정보는 그 타입의 외부에 존재해야 하는 것

특성정보를 다루는 표준적인 방법은 해당 특성정보를 템플릿 및 그 템플릿의 1개 이상의 특수화 버전에 넣는 것

반복자의 경우, 표준 라이브러리의 특성정보용 템플릿이 iterator_traits라는 이름으로 준비되어 있음

```c++
template<typename IterT>
struct iterator_traits;
```

특성정보는 항상 구조체로 구현하는 것이 관례

특성정보 클래스 : 특성정보를 구현하는 데 사용한 구조체

---

iterator_traits\<IterT\> 안에는 IterT 타입 각각에 대해 iterator_category라는 이름의 typedef 타입이 선언되어 있음

이렇게 선언된 typedef 타입이 IterT의 반복자 범주를 가리키는 것

iterator_traits 클래스는 이 반복자 범주를 두 부분으로 나누어 구현함

**첫 번째 부분은 사용자 정의 반복자 타입에 대한 구현**

사용자 정의 반복자 타입으로 하여금 iterator_category라는 이름의 typedef 타입을 내부에 가질 것을 요구사항으로 둠

이때 이 typedef 타입은 해당 태그 구조체에 대응되어야 함

예를 들어, deque의 반복자는 임의 접근 반복자이므로, deque 클래스에 쓸 수 있는 반복자는 다음과 같은 형태

```c++
template<...>
class deque {
public:
    class iterator {
    public:
        typedef random_access_iterator_tag iterator_category;
        ...
    };
    ...
};
```

이 iterator 클래스가 내부에 지닌 중첩 typedef 타입을 똑같이 재생한 것이 iterator_traits

```c++
template<typename IterT>
struct iterator_traits {
    typedef typename IterT::iterator_category iterator_category;
    ...
};
```

**두 번째 부분은 반복자가 포인터인 경우의 처리**

포인터 타입의 반복자를 지원하기 위해, iterator_traits는 포인터 타입에 대한 부분 템플릿 특수화(partial template specialization) 버전을 제공하고 있음

포인터의 동작 원리가 임의 접근 반복자와 똑같으므로, iterator_traits가 이런 식으로 지원하는 반복자 범주가 바로 임의 접근 반복자임

```c++
template<typename IterT>
struct iterator_traits<IterT*> {  // 기본제공 포인터 타입에 대한 부분 템플릿 특수화
    typedef random_access_iterator_tag iterator_category;
    ...
};
```

---

특성정보 클래스의 설계 및 구현 방법

- 다른 사람이 사용하도록 열어 주고 싶은 타입 관련 정보 확인(ex. 반복자라면 반복자 범주 등)
- 그 정보를 식별하기 위한 이름 선택(ex. iterator_category)
- 지원하고자 하는 타입 관련 정보를 담은 템플릿 및 그 템플릿의 특수화 버전(ex. iterator_traits) 제공

---

advance의 동작 원리는 똑같게 하고, 받아들이는 iterator_category 객체의 타입을 다르게 해서 오버로드 함수 구현

```c++
template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::random_access_iterator_tag) {
    iter += d;
}

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::bidirectional_iterator_tag) {
    if (d >= 0) { while (d--) ++iter; }
    else { while (d++) --iter; }
}

template<typename IterT, typename DistT>
void doAdvance(IterT& iter, DistT d, std::input_iterator_tag) {
    if (d < 0)
        throw std::out_of_range("Negative distance");
    while (d--) ++iter;
}
```

forward_iterator_tag는 input_iterator_tag로부터 상속을 받은 것이므로, input_iterator_tag를 매개변수로 받는 doAdvance는 순방향 반복자도 받을 수 있음

---

advance에서 오버로딩된 doAdvance 호출

컴파일러가 오버로딩 모호성 해결을 통해 적합한 버전을 호출할 수 있도록 반복자 범주 타입 객체를 맞추어 전달해야 함

```c++
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT d) {
    doAdvance(iter, d, typename std::iterator_traits<IterT>::iterator_category());
}
```

---

특성정보 클래스 사용 방법

- 작업자(worker) 역할을 맡을 함수 혹은 함수 템플릿(ex. doAdvance)을 특성정보 매개변수를 다르게 하여 오버로딩. 그리고 전달되는 해당 특성정보에 맞추어 각 오버로드 버전 구현
- 작업자를 호출하는 주작업자(master) 역할을 맡을 함수 혹은 함수 템플릿(ex. advance) 구현. 이때 특성정보 클래스에서 제공되는 정보를 넘겨서 작업자를 호출하도록 구현

---

특성정보는 C++ 표준 라이브러리에서 흔히 쓰임

iterator_traits는 iterator_category 말고도 제공하는 반복자 관련 정보가 네 가지나 됨

문자 타입에 대한 정보를 담고 있는 char_traits

숫자 타입에 대한 정보(표현 가능한 최소값 및 최대값 등)를 담고 있는 numeric_limits

---

**특성정보 클래스는 컴파일 도중에 사용할 수 있는 타입 관련 정보를 만들어 냄. 또한 특성정보 클래스는 템플릿 및 템플릿 특수 버전을 사용하여 구현함**

**함수 오버로딩 기법과 결합하여 특성정보 클래스를 사용하면, 컴파일 타임에 결정되는 타입별 if...else 점검문을 구사할 수 있음**

