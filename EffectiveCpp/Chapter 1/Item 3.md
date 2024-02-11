# Item 3: 낌새만 보이면 const를 들이대 보자!

**const**

의미적인 제약(const 키워드가 붙은 객체는 외부 변경을 불가능하게 한다)을 소스 코드 수준에서 붙임

컴파일러가 이 제약을 단단히 지켜줌

---

클래스 바깥

- 전역 혹은 네임스페이스 유효범위의 상수 선언(정의)
- 파일, 함수, 블록 유효범위에서 static으로 선언한 객체에도 const 붙일 수 있음

클래스 내부

- 정적 멤버 및 비정적 데이터 멤버 모두 상수로 선언 가능

포인터

- 포인터 자체를 상수로 지정
- 포인터가 가리키는 데이터를 상수로 지정

```c++
char greeting[] = "Hello";

// 비상수 포인터, 비상수 데이터
char *p = greeting;

// 비상수 포인터, 상수 데이터
const char *p = greeting;

// 상수 포인터, 비상수 데이터
char * const p = greeting;

// 상수 포인터, 상수 데이터
const char * const p = greeting;
```

`const`가 `*`의 왼쪽에 있으면 **포인터가 가리키는 대상**이 상수

`const`가 `*`의 오른쪽에 있으면 **포인터 자체**가 상수

---

**STL 반복자**는 포인터를 본뜬 것

변경 불가능한 객체를 가리키는 반복자(즉, const T* 포인터의 STL 대응물)가 필요하다면 `const_iterator` 사용

---

**함수 선언 시 const 사용**

함수 반환 값, 각각의 매개변수, 멤버 함수 앞, 함수 전체

함수 반환 값을 상수로 정하면 사용자측의 에러 돌발 상황을 줄일 수 있음

ex) 두 수의 곱에 대입 연산

```c++
class Rational { ... };

const Rational operator*(const Rational& lhs, const Rational& rhs);
```

훌륭한 사용자 정의 타입은 기본제공 타입과의 쓸데없는 비호환성을 피한다.

const 매개변수 → 가능한 한 항상 사용

매개변수 혹은 지역 객체를 수정할 수 없게 하는 것이 목적이라면 const로 선언하자.

### 상수 멤버 함수

멤버 함수에 붙는 const → 해당 멤버 함수가 상수 객체에 대해 호출될 함수라는 것

1. 클래스의 인터페이스를 이해하기 좋게 하기 위함
   - 객체를 변경할 수 있는 함수, 변경할 수 없는 함수
2. 상수 객체 사용 가능 → 코드 효율
   - 상수 객체에 대한 참조자(reference-to-const)로 객체 전달
   - 상수 멤버 함수가 있어야 함

---

const에 따른 멤버 함수 오버로딩

```c++
class TextBlock {
public:
    ...
    // 상수 객체에 대한 operator[]
    const char& operator[](std::size_t position) const
    { return text[position]; }
    
    // 비상수 객체에 대한 operator[]
    char& operator[](std::size_t position)
    { return text[position]; }
    
private:
    std::string text;
};
```

```c++
TextBlock tb("Hello");
std::cout << tb[0];  // 비상수 멤버 호출

const TextBlock ctb("World");
std::cout << ctb[0]; // 상수 멤버 호출
```

실제 프로그램에서 상수 객체가 생기는 경우 → 상수 객체에 대한 포인터 혹은 상수 객체에 대한 참조자로 객체가 전달될 때

```c++
void print(const TextBlock& ctb) {  // ctb는 상수 객체
    std::cout << ctb[0];  // 상수 멤버 호출
}
```

operator[]의 비상수 멤버는 char의 참조자를 반환해야 함

기본제공 타입을 반환하는 함수의 반환 값을 수정하는 일은 절대로 있을 수 없기 때문

#### 비트수준 상수성(물리적 상수성)

어떤 멤버 함수가 그 객체의 어떤 데이터 멤버도 건드리지 않아야(정적 멤버 제외) 그 멤버 함수가 const임을 인정하는 개념

C++에서 정의하고 있는 상수성이 비트수준 상수성

하지만 어떤 포인터가 가리키는 대상을 수정하는 멤버 함수들 중 상당수가 제대로 const로 동작하지 않는데도 비트수준 상수성 검사를 통과함

#### 논리적 상수성

위와 같은 상황을 보완하는 대체 개념

일부 몇 비트 정도는 바꿀 수 있되, 그것을 사용자측에서 알아채지 못하게만 하면 상수 멤버 자격이 있다는 것

mutable은 비정적 데이터 멤버를 비트수준 상수성의 족쇄에서 풀어줌 → 상수 멤버 함수 안에서도 수정 가능

### 상수 멤버 및 비상수 멤버 함수에서 코드 중복 현상을 피하는 방법

경계 검사, 접근정보 로깅, 내부자료 무결성 검증 등의 코드를 전부 operator[]의 상수/비상수 버전에 넣어 버리면 코드 중복 발생

안정성도 유지하면서 코드 중복을 피하는 방법은 비상수 operator[]가 상수 버전을 호출하도록 구현하는 것

```c++
class TextBlock {
public:
    ...
    const char& operator[](std::size_t position) const {
        ...
        return text[position];
    }
    
    char& operator[](std::size_t position) {
        return
            const_cast<char&>(
                static_cast<const TextBlock&>(*this)[position]
            );
    }
    ...
};
```

첫 번째 캐스팅은 *this에 const를 붙이는 캐스팅(비상수 operator[]에서 상수 버전을 호출하기 위함)

두 번째 캐스팅은 상수 operator[]의 반환 값에서 const를 떼어내는 캐스팅

상수 멤버에서 비상수 멤버 호출 X

---

const를 아끼지 말고 남발하자

- 포인터, 반복자
- 포인터/반복자/참조자가 가리키는 객체
- 함수의 매개변수 및 반환 타입
- 지역변수
- 멤버 함수

---

**const를 붙여 선언하면 컴파일러가 사용상의 에러를 잡아내는 데 도움을 준다.**

**컴파일러 쪽에서 보면 비트수준 상수성을 지켜야 하지만, 우리는 논리적인 상수성을 사용해서 프로그래밍 하자**

**상수 멤버 및 비상수 멤버 함수가 기능적으로 똑같게 구현되어 있을 경우 비상수 버전이 상수 버전을 호출하도록 해 중복을 피하자**

