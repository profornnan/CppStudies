# Item 24: 타입 변환이 모든 매개변수에 대해 적용되어야 한다면 비멤버 함수를 선언하자

클래스에서 암시적 타입 변환을 지원하는 것은 일반적으로 못된 생각이지만 예외가 있음

가장 흔한 예외 중 하나가 숫자 타입을 만들 때

유리수를 나타내는 클래스 → 정수에서 유리수로의 암시적 변환은 허용하자고 판단해도 크게 어이없거나 하진 않음

```c++
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);  // int에서 Rational로의 암시적 변환 허용을 위해 생성자에 일부러 explicit를 붙이지 않음
    int numerator() const;
    int denominator() const;
private:
    ...
};
```

operator*를 Rational 클래스 안에 구현

```c++
class Rational {
public:
    ...
    const Rational operator*(const Rational& rhs) const;
}
```

상수를 값으로 반환, 상수에 대한 참조자를 인자로 받음 → Item 3, Item 20, Item 21 참고

유리수 곱샘 쉽게 가능

```c++
Rational oneEighth(1, 8);
Rational oneHalf(1, 2);
Rational result = oneHalf * oneEighth;
result = result * oneEighth;
```

혼합형(mixed-mode) 수치 연산

```c++
result = oneHalf * 2;  // Success
result = 2 * oneHalf;  // Error
```

함수 형태로 변경

```c++
result = oneHalf.operator*(2);  // Success
result = 2.operator*(oneHalf);  // Error
```

정수 2에는 클래스 같은 것이 연관되어 있지 않기 때문에 operator* 멤버 함수도 있을 리가 없음

컴파일러는 아래처럼 호출할 수 있는 비멤버 버전의 operator\*(네임스페이스 혹은 전역 유효범위에 있는 operator\*)도 찾아봄

```c++
result = operator*(2, oneHalf);  // Error
```

int와 Rational을 취하는 비멤버 버전의 operator*가 없으므로 탐색은 실패하고 컴파일 에러 발생

---

암시적 타입 변환(implicit type conversion)

컴파일러는 아래와 같이 작성된 코드인 것처럼 처리함

```c++
const Rational temp(2);  // 2로부터 임시 Rational 객체 생성
result = oneHalf * temp;
```

컴파일러가 이렇게 동작한 것은 명시호출(explicit)로 선언되지 않은 생성자가 있기 때문임

Rational 생성자가 만약 명시호출 생성자였으면 다음의 코드 중 어느 쪽도 컴파일되지 않음

```c++
result = oneHalf * 2;  // Error! (명시호출 생성자에 의해) 2를 Rational로 바꿀 수 없음
result = 2 * oneHalf;  // Error
```

동작도 일관되게 유지하고 혼합형 수치 연산도 제대로 지원하는 것이 목적

암시적 타입 변환에 대해 매개변수가 먹혀들려면 매개변수 리스트에 들어 있아야 함

호출되는 멤버 함수를 갖고 있는(this가 가리키는) 객체에 해당하는 암시적 매개변수에는 암시적 변환이 먹히지 않음

혼합형 수치 연산 지원

operator*를 비멤버 함수로 만들어서, 컴파일러 쪽에서 모든 인자에 대해 암시적 타입 변환을 수행하도록 내버려 두는 것

```c++
class Rational {
    ...  // operator*가 없음
};

// 비멤버 함수
const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
}

Rational oneFourth(1, 4);
Rational result;

result = oneFourth * 2;  // Success
result = 2 * oneFourth;  // Success
```

operator*는 완전히 Rational의 public 인터페이스만을 써서 구현할 수 있기 때문에 Rational 클래스의 프렌드 함수로 두면 안 됨

멤버 함수의 반대는 프렌드 함수가 아니라 비멤버 함수

프렌드 함수는 피할 수 있으면 피하자

'멤버 함수이면 안 되니까'가 반드시 '프렌드 함수이어야 해'를 뜻하진 않음

템플릿 → Item 46 참고

---

**어떤 함수에 들어가는 모든 매개변수(this 포인터가 가리키는 객체도 포함해서)에 대해 타입 변환을 해 줄 필요가 있다면, 그 함수는 비멤버이어야 함**

