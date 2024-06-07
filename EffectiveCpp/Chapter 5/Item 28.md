# Item 28: 내부에서 사용하는 객체에 대한 '핸들'을 반환하는 코드는 되도록 피하자

Rectangle 클래스

사각형의 영역을 정의하는 꼭짓점을 별도의 구조체에 넣은 후에 Rectangle이 이 구조체를 가리키도록 구현

```c++
class Point {
public:
    Point(int x, int y);
    ...
    void setX(int newVal);
    void setY(int newVal);
    ...
};

struct RectData {
    Point ulhc;  // 좌측 상단
    Point lrhc;  // 우측 하단
};

class Rectangle {
    ...
private:
    std::tr1::shared_ptr<RectData> pData;
};
```

사용자 정의 타입 전달 시 값에 의한 전달보다 참조에 의한 전달방식을 쓰는 편이 더 효율적임 → Item 20 참고

```c++
class Rectangle {
public:
    ...
    Point& upperLeft() const { return pData->ulhc; }
    Point& lowerRight() const { return pData->lrhc; }
    ...
};
```

자기모순적인 코드

upperLeft와 lowerRight는 상수 멤버 함수

호출한 쪽은 은밀한 곳에 숨겨진 Point 데이터 멤버를 참조자로 끌어와 바꿀 수 있음

---

클래스 데이터 멤버는 아무리 숨겨봤자 그 멤버의 참조자를 반환하는 함수들의 최대 접근도에 따라 캡슐화 정도가 정해짐

ulhc와 lrhc는 private로 선언되어 있지만 실질적으로 public 멤버 → 이들의 참조자를 반환하는 upperLeft와 lowerRight 함수가 public 멤버 함수이기 때문

---

어떤 객체에서 호출한 상수 멤버 함수의 참조자 반환 값의 실제 데이터가 그 객체의 바깥에 저장되어 있다면, 이 함수의 호출부에서 그 데이터의 수정이 가능함

---

포인터나 반복자를 반환하도록 되어 있었다 해도 마찬가지

---

일반적인 수단으로 접근이 불가능한(protected 혹은 private로 선언된) 멤버 함수도 객체의 내부요소

이들에 대한 핸들(handle, 다른 객체에 손을 댈 수 있게 하는 매개자)도 반환하지 말아야 함

외부 공개가 차단된 멤버 함수에 대해, 이들의 포인터를 반환하는 멤버 함수를 만드는 일이 절대로 없어야 함

---

반환 타입에 const 키워드를 붙여서 해결

```c++
class Rectangle {
public:
    ...
    const Point& upperLeft() const { return pData->ulhc; }
    const Point& lowerRight() const { return pData->lrhc; }
    ...
};
```

사용자는 사각형을 정의하는 꼭짓점 쌍을 읽을 수는 있지만 쓸 수는 없게 됨

---

무효참조 핸들(dangling handle)

핸들이 있기는 하지만 그 핸들을 따라갔을 때 실제 객체의 데이터가 없는 것

객체의 내부에 대한 핸들을 반환하는 함수는 위험함

일단 바깥으로 떨어져 나간 핸들은 그 핸들이 참조하는 객체보다 더 오래 살 위험이 있음

---

핸들을 반환하는 멤버 함수를 절대로 두지 말라는 얘기는 아님

ex) operator[] 연산자는 string이나 vector 등의 클래스에서 개개의 원소를 참조할 수 있게 만드는 용도로 제공됨. 이 연산자는 내부적으로 해당 컨테이너에 들어 있는 개개의 원소 데이터에 대한 참조자를 반환하는 식으로 동작

이런 함수는 예외적인 것

---

**어떤 객체의 내부요소에 대한 핸들(참조자, 포인터, 반복자)을 반환하는 것은 되도록 피하자. 캡슐화 정도를 높이고, 상수 멤버 함수가 객체의 상수성을 유지한 채로 동작할 수 있도록 하며, 무효참조 핸들이 생기는 경우를 최소화할 수 있음**

