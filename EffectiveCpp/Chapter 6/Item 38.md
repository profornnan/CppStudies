# Item 38: "has-a(...는 ...를 가짐)" 혹은 "is-implemented-in-terms-of(...는 ...를 써서 구현됨)"를 모형화할 때는 객체 합성을 사용하자

합성(composition) : 어떤 타입의 객체들이 그와 다른 타입의 객체들을 포함하고 있을 경우에 성립하는 그 타입들 사이의 관계

포함된 객체들을 모아서 이들을 포함한 다른 객체를 합성한다는 뜻

```c++
class Address { ... };
class PhoneNumber { ... };
class Person {
public:
    ...
private:
    std::string name;
    Address address;
    PhoneNumber voiceNumber;
    PhoneNumber faxNumber;
};
```

Person 객체는 string, Address, PhoneNumber 객체로 이루어져 있음

레이어링(layering), 포함(containment), 통합(aggregation), 내장(embedding) 등

---

객체 합성

has-a(...는 ...를 가짐)

is-implemented-in-terms-of(...는 ...를 써서 구현됨)

뜻이 두 개인 이유는 SW 개발에서 대하는 영역(domain)이 두 가지이기 때문

응용 영역(application domain) : 일상생활에서 볼 수 있는 사물을 본 뜬 객체. 사람, 이동수단, 비디오 프레임 등

구현 영역(implementation domain) : 응용 영역에 속하지 않는 나머지들. 순수하게 시스템 구현만을 위한 인공물. 버퍼, 뮤텍스, 탐색트리 등

객체 합성이 응용 영역의 객체들 사이에서 일어나면 has-a 관계, 구현 영역에서 일어나면 is-implemented-in-terms-of 관계

---

Person 클래스가 나타내는 관계는 has-a 관계

---

is-a 관계와 is-implemented-in-terms-of 관계의 차이점

Set 템플릿 구현

C++ 라이브러리의 list 템플릿 사용

list 객체에 해당하는 사실들이 Set 객체에서도 통하는 게 아님 → Set이 list의 일종(is-a)이라는 명제는 참이 아님 → public 상속은 맞지 않음

Set 객체는 list 객체를 써서 구현되는(is-implemented-in-terms-of) 형태

```c++
template<class T>
class Set {
public:
    bool member(const T& item) const;
    void insert(const T& item);
    void remove(const T& item);
    std::size_t size() const;
private:
    std::list<T> rep;
};

template<typename T>
bool Set<T>::member(const T& item) const {
    return std::find(rep.begin(), rep.end(), item) != rep.end();
}

template<typename T>
void Set<T>::insert(const T& item) {
    if (!member(item)) rep.push_back(item);
}

template<typename T>
void Set<T>::remove(const T& item) {
    typename std::list<T>::iterator it = std::find(rep.begin(), rep.end(), item);
    if (it != rep.end()) rep.erase(it);
}

template<typename T>
std::size_t Set<T>::size() const {
    return rep.size();
}
```

Set과 list 사이의 관계는 is-a가 아니라 is-implemented-in-terms-of

---

**객체 합성(composition)의 의미는 public 상속이 가진 의미와 완전히 다름**

**응용 영역에서 객체 합성의 의미는 has-a(...는 ...를 가짐)**

**구현 영역에서 객체 합성의 의미는 is-implemented-in-terms-of(...는 ...를 써서 구현됨)**

