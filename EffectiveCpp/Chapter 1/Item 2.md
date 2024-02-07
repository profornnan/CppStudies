# Item 2: #define을 쓰려거든 const, enum, inline을 떠올리자

가급적 선행 처리자보다 컴파일러를 더 가까이 하자

```c++
#define ASPECT_RATIO 1.653
```

소스 코드가 컴파일러에게 넘어가기 전에 선행 처리자가 숫자 상수로 바꾸어 버리기 때문에 `ASPECT_RATIO`는 컴파일러가 쓰는 기호 테이블에 들어가지 않음

숫자 상수로 대체된 코드에서 컴파일 에러 발생 시 헷갈릴 수 있음

기호식 디버거(symbolic debugger)에서도 문제 발생 가능

**해결법 : 매크로 대신 상수 사용**

```c++
const double AspectRatio = 1.653;
```

상수 타입 데이터 → 컴파일러 눈에 보임. 기호 테이블에 들어감.

매크로 사용 시 목적 코드 안에 1.653의 사본이 등장 횟수만큼 들어가지만 상수 타입의 AspectRatio는 여러번 쓰이더라도 사본은 딱 한 개만 생성됨

---

`#define`을 상수로 교체 시 두 가지 경우 조심

**상수 포인터 정의**

- 포인터는 꼭 const로 선언
- 포인터가 가리키는 대상까지 const로 선언하는 것이 보통

```c++
const char * const authorName = "Scott Meyers";
```

문자열 상수 쓸 때 string 객체 사용이 더 좋음

```c++
const std::string authorName("Scott Meyers");
```

**클래스 멤버로 상수 정의**

상수 유효범위를 클래스로 한정 → 멤버로 만들어야 함

상수의 사본 개수가 한 개를 넘지 못하게 하려면 정적(static) 멤버로 만들어야 함

```c++
class GamePlayer {
private:
    static const int NumTurns = 5; // 상수 선언
    int scores[NumTurns];          // 상수 사용
}
```

```c++
const int GamePlayer::NumTurns;  // NumTurns 정의
```

클래스 상수의 정의는 선언될 당시에 바로 초기화되기 때문에 헤더 파일이 아닌 구현 파일에 둔다.

`#define`은 유효범위가 없음 → 클래스 상수 정의 시 사용 불가. 캡슐화 X

위의 문법이 먹히지 않는 컴파일러 사용시 초기값을 상수 정의 시점에 주자.

---

해당 클래스를 컴파일하는 도중에 클래스 상수값이 필요한 경우

ex) 배열 멤버 선언시 (배열 크기)

**나열자 둔갑술(enum hack)**

나열자(enumerator) 타입의 값은 int가 놓일 곳에도 쓸 수 있다.

```c++
class GamePlayer {
private:
    enum { NumTurns = 5 };
    
    int scores[NumTurns];
    ...
}
```

나열자 둔갑술 동작 방식은 `const`보다 `#define`에 더 가깝다.

- 주소 X, 참조자 X
- enum은 쓸데없는 메모리 할당 X

나열자 둔갑술은 템플릿 메타프로그래밍의 핵심 기법

---

**매크로 함수** → 사용 X

함수 호출 오버헤드를 일으키지 않는 매크로 구현

매크로 작성 시 인자마다 반드시 괄호를 씌워야 함

```c++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
```

비교를 통해 처리한 결과가 어떤 것인지에 따라 달라짐

**인라인 함수에 대한 템플릿**

기존 매크로 효율 그대로 유지

정규 함수의 모든 동작방식 및 타입 안전성까지 취할 수 있음

```c++
template<typename T>
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```

동일 계열 함수군을 만들어 냄

동일한 타입의 객체 두 개를 인자로 받고 둘 중 큰 것을 f에 넘겨서 호출하는 구조

유효 범위 및 접근 규칙 그대로 따라감

---

**단순한 상수 → #define보다 const 객체 혹은 enum 사용**

**함수처럼 쓰이는 매크로 → #define 매크로보다 인라인 함수 사용**

