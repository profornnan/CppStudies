# Item 45: "호환되는 모든 타입"을 받아들이는 데는 멤버 함수 템플릿이 직방!

스마트 포인터는 그냥 포인터처럼 동작하면서도 포인터가 주지 못하는 기능을 덤으로 갖고 있는 객체

STL 컨테이너의 반복자도 스마트 포인터나 마찬가지임

포인터에도 스마트 포인터로 대신할 수 없는 특징이 있음 → 암시적 변환(implicit conversion) 지원

```c++
class Top { ... };
class Middle: public Top { ... };
class Bottom: public Middle { ... };
Top *pt1 = new Middle;
Top *pt2 = new Bottom;
const Top *pct2 = pt1;
```

이런 식의 타입 변환을 사용자 정의 스마트 포인터를 써서 흉내 내려면 무척 까다로움

같은 템플릿으로부터 만들어진 다른 인스턴스들 사이에는 어떤 관계도 없기 때문에, 변환이 되도록 직접 프로그램을 만들어야 함

우리가 원하는 생성자의 개수는 무제한 → 생성자를 만들어내는 템플릿 사용

멤버 함수 템플릿(member function template) : 어떤 클래스의 멤버 함수를 찍어내는 템플릿

```c++
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other);
    ...
};
```

일반화된 복사 생성자를 만들기 위해 마련한 멤버 템플릿

모든 T 타입 및 모든 U 타입에 대해서, SmartPtr\<T\> 객체가 SmartPtr\<U\>로부터 생성될 수 있음

일반화 복사 생성자(generalized copy constructor) : 같은 템플릿을 써서 인스턴스화되지만 타입이 다른 타입의 객체로부터 원하는 객체를 만들어 주는 생성자

---

SmartPtr이 get 멤버 함수를 통해 해당 스마트 포인터 객체에 자체적으로 담긴 기본제공 포인터의 사본을 반환한다면, 이것을 이용해서 생성자 템플릿에 원하는 타입 변환 제약을 줄 수 있음

```c++
template<typename T>
class SmartPtr {
public:
    template<typename U>
    SmartPtr(const SmartPtr<U>& other): heldPtr(other.get()) { ... }
    T* get() const { return heldPtr; }
    ...
private:
    T *heldPtr;
};
```

멤버 초기화 리스트를 사용해서, SmartPtr\<T\>의 데이터 멤버인 T\* 타입의 포인터를 SmartPtr\<U\>에 들어 있는 U\* 타입의 포인터로 초기화

U\*에서 T\*로 진행되는 암시적 변환이 가능할 때만 컴파일 에러가 나지 않음

---

대입 연산에도 멤버 함수 템플릿이 활용됨

ex) TR1의 shared_ptr(Item 13 참고) 클래스 템플릿은 호환되는 모든 기본제공 포인터, tr1::shared_ptr, auto_ptr, tr1::weak_ptr(Item 54 참고) 객체들로부터 생성자 호출이 가능한데다가, 이들 중 tr1::weak_ptr을 제외한 나머지를 대입 연산에 쓸 수 있도록 만들어져 있음

---

멤버 함수 템플릿은 C++ 언어의 기본 규칙까지 바꾸지는 않음

'복사 생성자가 필요한데 프로그래머가 직접 선언하지 않으면 컴파일러가 자동으로 하나 만든다.'

일반화 복사 생성자(멤버 템플릿)는 보통의 복사 생성자가 아님 → 보통의 복사 생성자까지 직접 선언해야 함

대입 연산자도 마찬가지

tr1::shared_ptr

```c++
template<class T> class shared_ptr {
public:
    shared_ptr(shared_ptr const& r);  // 복사 생성자
    template<class Y> shared_ptr(shared_ptr<Y> const& r);  // 일반화 복사 생성자
    shared_ptr& operator=(shared_ptr const& r);  // 복사 대입 연산자
    template<class Y> shared_ptr& operator=(shared_ptr<Y> const& r);  // 일반화 복사 대입 연산자
    ...
};
```

---

**호환되는 모든 타입을 받아들이는 멤버 함수를 만들려면 멤버 함수 템플릿 사용**

**일반화된 복사 생성 연산과 일반화된 대입 연산을 위해 멤버 템플릿을 선언했다 하더라도, 보통의 복사 생성자와 복사 대입 연산자는 여전히 직접 선언해야 함**

