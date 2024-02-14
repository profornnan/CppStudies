# Item 6: 컴파일러가 만들어낸 함수가 필요 없으면 확실히 이들의 사용을 금해 버리자

일반적인 경우, 어떤 클래스에서 특정한 종류의 기능을 지원하지 않도록 하는 방법은 그런 기능을 제공하는 함수를 선언하지 않는 것

이 전략은 복사 생성자와 복사 대입 연산자에 대해서는 해당사항이 없음

복사 생성자 및 복사 대입 연산자를 private 멤버로 선언 → 외부 호출 차단

클래스의 멤버 함수 및 friend 함수가 호출할 수 없도록 일부러 정의(구현)하지 않는 방법 사용

정의되지 않은 함수 호출 시 링크 시점 에러 발생

C++의 iostream 라이브러리에 속한 몇몇 클래스에서도 복사 방지책으로 사용함

```c++
class HomeForSale {
public:
    ...
private:
    ...
    HomeForSale(const HomeForSale&);
    HomeForSale& operator=(const HomeForSale&);
};
```

링크 시점 에러를 컴파일 시점 에러로 옮기는 것이 더 좋음

복사 생성자와 복사 대입 연산자를 별도의 기본 클래스에 private로 선언하고, 이것으로부터 HomeForSale을 파생시키기

별도의 기본 클래스는 복사 방지를 맡는다는 의미 부여

```c++
class Uncopyable {
protected:
    Uncopyable() {}
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);
    Uncopyable& operator=(const Uncopyable&);
};

class HomeForSale: private Uncopyable {
    ...
};
```

HomeForSale 객체의 복사를 시도하려고 할 때 컴파일러는 HomeForSale 클래스만의 복사 생성자와 복사 대입 연산자를 만들려고 할 것임

Uncopyable로부터의 상속은 public일 필요 없음

Uncopyable의 소멸자는 가상 소멸자가 아니어도 됨

---

**컴파일러에서 자동으로 제공하는 기능을 허용치 않으려면, 대응되는 멤버 함수를 private로 선언한 후에 구현은 하지 않은 채로 두자**

**Uncopyable과 비슷한 기본 클래스를 쓰는 것도 한 방법**

