# Item 33: 상속된 이름을 숨기는 일은 피하자

유효범위(scope)

```c++
int x;  // 전역 변수

void someFunc() {
    double x;  // 지역 변수
    std::cin >> x;
}
```

값을 읽어 x에 넣는 문장에서 실제로 참조하는 x는 지역 변수 x

안쪽 유효범위에 있는 이름이 바깥쪽 유효범위에 있는 이름을 가리기 때문

C++의 이름 가리기 규칙은 이름을 가려버림

겹치는 이름들의 타입이 같으냐 다르냐에 대한 부분은 신경도 안 씀

---

기본 클래스에 속해 있는 것(멤버 함수, typedef, 데이터 멤버)을 파생 클래스 멤버 함수 안에서 참조하는 문장이 있으면 컴파일러는 이 참조 대상을 바로 찾아낼 수 있음

파생 클래스의 유효범위가 기본 클래스의 유효범위 안에 중첩되어 있기 때문

```c++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf2();
    void mf3();
    ...
};

class Derived: public Base {
public:
    virtual void mf1();
    void mf4();
    ...
};
```

mf4가 파생 클래스에서 다음과 같이 구현되어 있다고 가정

```c++
void Derived::mf4() {
    ...
    mf2();
    ...
}
```

이름의 출처 파악을 위해, 컴파일러는 mf2라는 이름이 붙은 것의 선언문이 들어 있는 유효범위를 탐색하는 방법을 씀

지역 유효범위(mf4의 유효범위) 내부에는 mf2라 불리는 어떤 것도 선언된 게 없음

mf4의 유효범위를 바깥에서 감싸고 있는 유효범위를 찾음 → Derived 클래스의 유효범위

여전히 mf2라는 이름을 가진 것이 보이지 않으므로, 컴파일러는 Base 클래스의 유효범위로 옮겨 감

여기서 컴파일러는 mf2라는 이름이 붙은 것을 찾아내고, 탐색이 끝남

만약 Base 안에 mf2가 없으면 계속 탐색이 진행되는데, 우선 Base를 둘러싸고 있는 네임스페이스가 있으면 그쪽부터 탐색을 시작해서, 마지막엔 전역 유효범위까지 감

위에서 말한 탐색 과정은 유효범위에서 컴파일러가 이름을 찾을 때 실제로 정확히 일어나는 과정이지만, C++의 이름 탐색 과정을 모두 설명한 것은 아님

---

mf1 및 mf3을 오버로드하고, mf3의 오버로드 버전을 Derived에 추가

```c++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived: public Base {
public:
    virtual void mf1();
    void mf3();
    void mf4();
    ...
};
```

기본 클래스에 있는 함수들 중에 mf1 및 mf3이라는 이름이 붙은 것은 모두 파생 클래스에 들어 있는 mf1 및 mf3에 의해 가려짐

이름 탐색의 시점에서 보면, Base::mf1과 Base::mf3은 Derived가 상속한 것이 아니게 됨

```c++
Derived d;
int x;
...
d.mf1();  // 좋음. Derived::mf1 호출
d.mf1(x);  // 에러. Derived::mf1이 Base::mf1을 가림
d.mf2();  // 좋음. Base::mf2 호출
d.mf3();  // 좋음. Derived::mf3 호출
d.mf3(x);  // 에러. Derived::mf3이 Base::mf3을 가림
```

이름 가리기는 기본 클래스와 파생 클래스에 있는 이름이 같은 함수들이 받아들이는 매개변수 타입, 가상 함수인지 비가상 함수인지의 여부에 상관없이 이름이 가려짐

어떤 라이브러리 혹은 응용프로그램 프레임워크를 이용하여 파생 클래스를 하나 만들 때, 멀리 떨어져 있는 기본 클래스로부터 오버로드 버전을 상속시키는 경우를 막겠다는 것

public 상속을 쓰면서 기본 클래스의 오버로드 함수를 상속받지 않겠다는 것도 엄연히 is-a 관계 위반임

C++의 상속된 이름 가리기를 무시하고 싶은 경우가 거의 대부분

가려진 이름은 using 선언을 써서 끄집어낼 수 있음

```c++
class Base {
private:
    int x;
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    virtual void mf2();
    void mf3();
    void mf3(double);
    ...
};

class Derived: public Base {
public:
    using Base::mf1;
    using Base::mf3;
    virtual void mf1();
    void mf3();
    void mf4();
    ...
};
```

Base에 있는 것들 중 mf1과 mf3을 이름으로 가진 것들을 Derived의 유효범위에서 볼 수 있도록 만듦

예상했던 대로 돌아감

어떤 기본 클래스로부터 상속을 받으려고 하는데, 오버로드된 함수가 그 클래스에 들어 있고 이 함수들 중 몇 개만 재정의(오버라이드)하고 싶다면, 각 이름에 대해 using 선언을 붙여 주어야 함

이렇게 하지 않으면 이름이 가려져 버림

---

기본 클래스가 가진 함수를 전부 상속했으면 하는 것이 아닌 경우도 있긴 함

private 상속(Item 39 참고)을 사용한다면 이 경우가 말이 될 수 있음

Derived가 Base로부터 private 상속이 이루어졌다고 가정

Derived가 상속했으면 하는 mf1 함수는 매개변수가 없는 버전 하나

using 선언을 내리면 그 이름에 해당되는 것들이 모두 파생 클래스로 내려가 버리기 때문에 using 선언으로 해결할 수 없음

전달 함수(forwarding function) 만들기

```c++
class Base {
public:
    virtual void mf1() = 0;
    virtual void mf1(int);
    ...
};

class Derived: private Base {
public:
    virtual void mf1() { Base::mf1(); }  // 전달 함수. 암시적으로 인라인 함수가 됨(Item 30 참고)
    ...
};

...
Derived d;
int x;
d.mf1();  // 좋음. Derived::mf1(매개변수 없는 버전) 호출
d.mf1(x);  // 에러. Base::mf1()은 가려져 있음
```

---

**파생 클래스의 이름은 기본 클래스의 이름을 가림. public 상속에서는 이런 이름 가림 현상은 바람직하지 않음**

**가려진 이름을 다시 볼 수 있게 하는 방법으로, using 선언 혹은 전달 함수를 쓸 수 있음**

