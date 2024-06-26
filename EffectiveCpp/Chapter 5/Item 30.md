# Item 30: 인라인 함수는 미주알고주알 따져서 이해해 두자

인라인 함수는 훌륭한 아이디어

함수처럼 보이고 함수처럼 동작

매크로보다 훨씬 안전하고 쓰기 좋음 → Item 2 참고

함수 호출 시 발생하는 오버헤드도 걱정할 필요 없음

---

인라인 함수 사용 시 컴파일러가 함수 본문에 대해 문맥별(context-specific) 최적화를 걸기가 용이해짐

---

인라인 함수의 아이디어는 함수 호출문을 그 함수의 본문으로 바꿔치기하자는 것이라서 목적 코드의 크기가 커짐

페이징 횟수가 늘어나고, 명령어 캐시 적중률이 떨어질 가능성도 높아짐

---

본문 길이가 굉장히 짧은 인라인 함수 사용 시, 함수 본문에 대해 만들어지는 코드의 크기가 함수 호출문에 대해 만들어지는 코드보다 작아질 수도 있음

목적 코드의 크기도 작아지며 명령어 캐시 적중률도 높아짐

---

inline은 컴파일러에 요청을 하는 것이지 명령이 아님

inline을 붙이지 않아도 그냥 눈치껏 되는 경우도 있고 명시적으로 할 수도 있음

---

암시적인 방법

클래스 정의 안에 함수를 바로 정의해 넣으면 컴파일러는 그 함수를 인라인 함수 후보로 찍음

```c++
class Person {
public:
    ...
    int age() const { return theAge; }  // 암시적인 인라인 요청
    ...
private:
    int theAge;
};
```

이런 함수는 대개 멤버 함수이지만, 프렌드 함수도 클래스 내부에서 정의될 수 있음 → Item 46 참고

---

명시적인 방법

함수 정의 앞에 inline 키워드를 붙이는 것

ex) 표준 라이브러리의 max 템플릿

```c++
template<typename T>
inline const T& std::max(const T& a, const T& b) { return a < b ? b : a; }
```

---

인라인 함수는 대체적으로 헤더 파일에 들어 있어야 하는게 맞음

대부분의 빌드 환경에서 인라인을 컴파일 도중에 수행하기 때문

인라인 함수 호출을 그 함수의 본문으로 바꿔치기하려면, 일단 컴파일러는 그 함수가 어떤 형태인지 알고 있어야 함

템플릿도 대체적으로 헤더 파일에 들어 있어야 맞음

템플릿이 사용되는 부분에서 해당 템플릿을 인스턴스로 만들려면 그것이 어떻게 생겼는지를 컴파일러가 알아야 하기 때문

템플릿 인스턴스화는 인라인과 완전히 별개

템플릿으로부터 만들어지는 모든 함수가 인라인 함수였으면 싶은 경우에 그 템플릿에 inline을 붙여 선언

함수 템플릿이 굳이 인라인될 이유가 없다면 그 템플릿을 인라인 함수로 선언하지 않아도 됨

---

inline은 컴파일러 선에서 무시할 수 있는 요청임

복잡한 함수는 절대로 인라인 확장의 대상에 넣지 않음

가상 함수 호출 같은 것은 절대로 인라인해 주지 않음

virtual → 어떤 함수를 호출할지 결정하는 작업을 실행 중에 함

inline → 함수 호출 위치에 호출된 함수를 끼워 넣는 작업을 프로그램 실행 전에 함

---

확실한 인라인 함수도 어떻게 호출하느냐에 따라 인라인되기도 하고 안 되기도 함

ex) 어떤 인라인 함수의 주소를 취하는 코드가 있으면, 컴파일러는 아웃라인 함수를 본문에 만들 수밖에 없음

---

생성자와 소멸자는 인라인하기에 그리 좋지 않은 함수

C++는 객체가 생성되고 소멸될 때 일어나는 일들에 대해 여러 가지 보장을 준비해 놓았음

new를 하면 동적으로 만들어지는 객체를 생성자가 자동으로 초기화

delete를 하면 소멸자가 호출됨

C++는 무엇을 해야 하는지는 정해 두었지만 어떻게 해야 하는지는 정하지 않았음

이런 일을 가능하게 하는 어떤 코드가 프로그램에 포함되어야 하고, 이 코드(컴파일러가 만들어서 컴파일 도중에 소스 코드에 삽입하는 코드)가 소스 코드 어딘가에 들어가 있어야 함

그 장소가 생성자와 소멸자일 수도 있음

인라인이 난감해짐

---

라이브러리 설계 시 함수를 inline으로 선언할 때 그 영향에 대해 많은 고민을 해야 함

사용자의 눈에 뻔히 보이는 인라인 함수에 대해서는 라이브러리 차원에서 바이너리 업그레이드를 제공할 수 없기 때문

---

대부분의 디버거가 인라인 함수를 무척이나 곤란해 함

있지도 않은 함수에 중단점을 걸 수 없음

---

우선, 아무것도 인라인하지 말자

꼭 인라인해야 하는 함수(Item 46 참고) 혹은 정말 단순한 함수(예시로 나온 Person::age 같은 함수)에 한해서만 인라인 함수로 선언

인라인을 주의해서 사용하는 버릇을 들이자

---

가장 중요한 것은 함수

함수 자체를 똑바로 만들자

---

**함수 인라인은 작고, 자주 호출되는 함수에 대해서만 하는 것으로 묶어두자. 이렇게 하면 디버깅 및 라이브러리의 바이너리 업그레이드가 용이해지고, 자칫 생길 수 있는 코드 부풀림 현상이 최소화되며, 프로그램의 속력이 더 빨라질 수 있는 여지가 최고로 많아짐**

**함수 템플릿이 대개 헤더 파일에 들어간다는 일반적인 부분만 생각해서 이를 inline으로 선언하면 안 됨**

