# Item 11: operator=에서는 자기대입에 대한 처리가 빠지지 않도록 하자

자기대입 : 어떤 객체가 자기 자신에 대한 대입 연산자를 적용하는 것

자기대입이 생기는 이유 : 여러 곳에서 하나의 객체를 참조(중복참조, aliasing)

같은 타입으로 만들어진 객체 여러 개를 참조자 혹은 포인터로 물어 놓고 동작하는 코드 작성 시 같은 객체가 사용될 가능성을 고려하자

자원 관리 객체들이 복사될 때 조심해야 하는 것이 대입 연산자

자기대입에 대해 안전하게 동작해야 함

```c++
class Bitmap { ... };

class Widget {
    ...
private:
    Bitmap *pb;  // 힙에 할당한 객체를 가리키는 포인터
};

Widget& Widget::operator=(const Widget& rhs) {  // 안전하지 않게 구현됨
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

*this와 rhs가 같은 객체라면 delete 연산자가 rhs에도 적용됨

일치성 검사로 해결

```c++
Widget& Widget::operator=(const Widget& rhs) {
    if (this == &rhs) return *this;
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```

new Bitmap에서 예외 발생 시 삭제된 Bitmap을 가리키는 포인터만 남게 됨

**많은 경우에 문장 순서를 세심하게 바꾸는 것만으로 예외에 안전한 코드가 만들어짐**

```c++
Widget& Widget::operator=(const Widget& rhs) {
    Bitmap *pOrig = pb;
    pb = new Bitmap(*rhs.pb);
    delete pOrig;
    return *this;
}
```

pb가 가리키는 객체를 복사한 직후에 삭제

예외에 안전

---

복사 후 맞바꾸기(copy and swap) 기법 (Item 29)

```c++
class Widget {
    ...
    void swap(Widget& rhs);  // Item 29 참고
    ...
};

Widget& Widget::operator=(const Widget& rhs) {
    Widget temp(rhs);
    swap(temp);
    return *this;
}
```

rhs의 데이터에 대해 사본 생성 후 *this의 데이터를 그 사본의 것과 맞바꿈

---

C++의 두 가지 특징 활용

1. 클래스의 복사 대입 연산자는 인자를 값으로 취하도록 선언 가능
2. 값에 의한 전달 수행 시 전달된 대상의 사본이 생성됨

```c++
Widget& Widget::operator=(Widget rhs) {
    swap(rhs);
    return *this;
}
```

---

**operator= 구현 시 자기대입에 대한 처리 하기 → 원본 객체와 복사대상 객체 주소 비교, 문장 순서 조정, 복사 후 맞바꾸기**

**두 개 이상의 객체에 대해 동작하는 함수가 있다면, 이 함수에 넘겨지는 객체들이 같은 객체인 경우에 정확하게 동작하는지 확인**

