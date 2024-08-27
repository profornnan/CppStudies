# Item 46: 타입 변환이 바람직할 경우에는 비멤버 함수를 클래스 템플릿 안에 정의해 두자

모든 매개변수에 대해 암시적 타입 변환이 되도록 만들기 위해서는 비멤버 함수밖에 방법이 없음 → Item 24 참고

Rational 클래스와 operator* 함수를 템플릿으로

```c++
template<typename T>
class Rational {
public:
    Rational(const T& numerator = 0, const T& denominator = 1);
    const T numerator() const;
    const T denominator() const;
    ...
};

template<typename T>
const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs) { ... }
```

```c++
Rational<int> oneHalf(1, 2);
Rational<int> result = oneHalf * 2;  // Error
```

컴파일이 되지 않음

Item 24에서는 호출하려고 하는 함수가 무엇인지를 컴파일러가 알고 있지만, 지금 경우에는 어떤 함수를 호출하려는지에 대해 컴파일러로서는 아는 바가 전혀 없음

단지, 컴파일러는 operator\*라는 이름의 템플릿으로부터 인스턴스화할 함수를 결정하기 위해 온갖 계산을 동원할 뿐임

템플릿 인자 추론(template argument deducation) 과정에서는 생성자 호출을 통한 암시적 타입 변환이 고려되지 않음

---

클래스 템플릿 안에 프렌드 함수를 넣어 두면 함수 템플릿으로서의 성격을 주지 않고 특정한 함수 하나를 나타낼 수 있음

클래스 템플릿은 템플릿 인자 추론 과정에 좌우되지 않으므로, T의 정확한 정보는 Rational\<T\> 클래스가 인스턴스화될 당시에 바로 알 수 있음

템플릿 인자 추론은 함수 템플릿에만 적용되는 과정

---

클래스 템플릿 내부에서는 템플릿의 이름을 그 템플릿 및 매개변수의 줄임말로 쓸 수 있음

operator\* 함수의 본문을 선언부와 붙이기

```c++
template<typename T>
class Rational {
public:
    ...
    friend const Rational operator*(const Rational& lhs, const Rational& rhs) {
        return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
    }
};
```

프렌드 함수를 선언하긴 했지만, 클래스의 public 영역이 아닌 부분에 접근하는 것과 프렌드 권한은 아무런 상관이 없음

클래스 안에 비멤버 함수를 선언하는 유일한 방법이 프렌드였을 뿐임

---

클래스 안에 정의된 함수는 암시적으로 인라인으로 선언됨 → Item 30 참고

클래스의 바깥에서 정의된 도우미 함수만 호출하는 식으로 operator\*를 구현하면 암시적 인라인 선언의 영향을 최소화할 수도 있음

"프렌드 함수는 도우미만 호출하게 만들기" 방법

대다수의 컴파일러에서 템플릿 정의를 헤더 파일에 전부 넣을 것을 사실상 강제로 강요하다시피 하고 있음

---

**모든 매개변수에 대해 암시적 타입 변환을 지원하는 템플릿과 관계가 있는 함수를 제공하는 클래스 템플릿을 만들려고 한다면, 이런 함수는 클래스 템플릿 안에 프렌드 함수로서 정의하자**

