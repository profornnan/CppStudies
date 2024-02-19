# Item 10: 대입 연산자는 *this의 참조자를 반환하게 하자

C++의 대입 연산은 여러 개가 사슬처럼 엮일 수 있음

우측 연관 연산

```c++
int x, y, z;
x = y = z = 15;
x = (y = (z = 15));
```

대입 연산자가 좌변 인자에 대한 참조자를 반환 → 일종의 관례

```c++
class Widget {
public:
    ...
    Widget& operator=(const Widget& rhs) {
        ...
        return *this;
    }
    ...
};
```

좌변 객체의 참조자를 반환하게 만들자는 규약은 모든 형태의 대입 연산자에서 지켜져야 함

```c++
class Widget {
public:
    ...
    Widget& operator+=(const Widget& rhs) {
        ...
        return *this;
    }
    
    Widget& operator=(int rhs) {
        ...
        return *this;
    }
    ...
};
```

대입 연산자의 매개변수 타입이 일반적이지 않은 경우에도 동일한 규약 적용

---

**대입 연산자는 *this의 참조자를 반환하도록 만들자**

