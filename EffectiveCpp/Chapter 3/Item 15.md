# Item 15: 자원 관리 클래스에서 관리되는 자원은 외부에서 접근할 수 있도록 하자

자원 관리 클래스는 실수로 터질 수 있는 자원 누출을 막아주는 역할을 함

API가 자원을 직접 참조하도록 만들어져 있어서 실제 자원을 직접 건드려야 하는 경우가 있음

Item 13에서 팩토리 함수를 호출한 결과(포인터)를 담기 위해 스마트 포인터 사용

```c++
std::tr1::shared_ptr<Investment> pInv(createInvestment());
```

사용하려는 함수

```c++
int daysHeld(const Investment *pi);  // 투자금이 유입된 이후로 경과한 날수
```

RAII 클래스의 객체(tr1::shared_ptr)를 그 객체가 감싸고 있는 실제 자원(Investment*)으로 변환할 방법이 필요함

명시적 변환(explicit conversion)과 암시적 변환(implicit conversion)이 있음

tr1::shared_ptr 및 auto_ptr은 명시적 변환을 수행하는 get 멤버 함수 제공

```c++
int days = daysHeld(pInv.get());
```

tr1::shared_ptr과 auto_ptr은 포인터 역참조 연산자(operator-> 및 operator*)도 오버로딩 하고 있음

→ 자신이 관리하는 실제 포인터에 대한 암시적 변환 가능

---

자원 접근을 매끄럽게 할 수 있도록 RAII 클래스에서 암시적 변환 함수를 제공하는 경우

ex) 하부 수준 C API로 직접 조작이 가능한 폰트를 RAII 클래스로 둘러싸서 사용하는 경우

```c++
FontHandle getFont();  // C API에서 가져온 함수
void releaseFont(FontHandle fh);  // C API에서 가져온 함수

class Font {  // RAII 클래스
public:
    explicit Font(FontHandle fh): f(fh) {}  // 자원 획득
    ~Font() { releaseFont(f); }

private:
    FontHandle f;  // 실제 폰트 자원
};
```

Font 클래스에서 명시적 변환 함수로 get 제공 가능

```c++
class Font {
public:
    ...
    FontHandle get() const { return f; }  // 명시적 변환 함수
    ...
};
```

하부 수준 API를 쓰고 싶을 때마다 get을 호출해야 함

```c++
void changeFontSize(FontHandle f, int newSize);  // 폰트 API의 일부

Font f(getFont());
int newFontSize;
...
changeFontSize(f.get(), newFontSize);  // Font에서 FontHandle로 명시적으로 변경 후 넘김
```

암시적 변환 함수 제공

```c++
class Font {
public:
    ...
    operator FontHandle() const { return f; }  // 암시적 변환 함수
    ...
};
```

C API 사용이 훨씬 쉽고 자연스러워짐

```c++
Font f(getFont());
int newFontSize;
...
changeFontSize(f, newFontSize);  // Font에서 FontHandle로 암시적 변환 수행
```

암시적 변환이 들어가면 실수를 저지를 여지가 많아짐

Font를 쓰려고 한 부분에서 원하지도 않았는데 FontHandle로 바뀔 수 있음

---

RAII 클래스를 실제 자원으로 바꾸는 방법으로 암시적 변환보다는 get 등의 명시적 변환 함수를 제공하는 쪽이 나을 때가 많지만 암시적 타입 변환에서 생기는 사용 시의 자연스러움이 빛을 발하는 경우도 있음

---

RAII 클래스는 데이터 은닉이 목적이 아님

자원 해제가 실수 없이 이루어지도록 하면 됨

캡슐화가 꼭 필요한 것은 아님

tr1::shared_ptr은 엄격한 캡슐화와 느슨한 캡슐화 동시에 지원

참조 카운팅 메커니즘에 필요한 장치들은 모두 캡슐화

자신이 관리하는 포인터는 쉽게 접근 가능

---

실제 자원을 직접 접근해야 하는 기존 API들도 많기 때문에, RAII 클래스를 만들 때는 그 클래스가 관리하는 자원을 얻을 수 있는 방법을 열어줘야 함

자원 접근은 명시적 변환 혹은 암시적 변환을 통해 가능함. 안전성만 따지면 명시적 변환이 대체적으로 더 낫지만, 고객 편의성을 놓고 보면 암시적 변환이 괜찮음

