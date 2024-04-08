# Item 21: 함수에서 객체를 반환해야 할 경우에 참조자를 반환하려고 들지 말자

실제로 있지도 않는 객체의 참조자를 넘기지 않도록 주의

---

```c++
class Rational {
public:
    Rational(int numerator = 0, int denominator = 1);
    ...
private:
    int n, d;
    friend const Rational operator*(const Rational& lhs, const Rational& rhs);
};
```

유리수를 나타내는 클래스

---

새로운 객체를 반환해야 하는 함수를 작성하는 방법 → 새로운 객체를 반환하게 만드는 것

```c++
inline const Rational operator*(const Rational& lhs, const Rational& rhs) {
    return Rational(lhs.n * rhs.n, lhs.d * rhs.d);
}
```

반환 값을 생성하고 소멸시키는 비용이 들어 있음 → 올바른 동작에 지불되는 작은 비용

컴파일러 구현자들이 가시적인 동작 변경을 가하지 않고도 기존 코드의 수행 성능을 높이는 최적화를 적용할 수 있도록 배려해 둠

몇몇 조건하에서는 이 최적화 메커니즘에 의해 operator*의 반환 값에 대한 생성과 소멸 동작이 안전하게 제거될 수 있음

반환 값 최적화(return value optimization: RVO)

---

참조자를 반환할 것인가 아니면 객체를 반환할 것인가 → 어떤 선택을 하든 올바른 동작이 이루어지도록 만들어야 함

---

**지역 스택 객체에 대한 포인터나 참조자를 반환하는 일, 혹은 힙에 할당된 객체에 대한 참조자를 반환하는 일, 또는 지역 정적 객체에 대한 포인터나 참조자를 반환하는 일은 그런 객체가 두 개 이상 필요해질 가능성이 있다면 절대로 하지 말자.**

