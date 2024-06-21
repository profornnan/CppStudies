# Item 34: 인터페이스 상속과 구현 상속의 차이를 제대로 파악하고 구별하자

public 상속은 두 가지로 나뉨 → 함수 인터페이스의 상속, 함수 구현의 상속

인터페이스 상속 및 구현 상속의 차이는 함수 선언(function declaration) 및 함수 정의(function definition)의 차이와 맥을 같이 함

---

멤버 함수의 인터페이스(선언)만을 파생 클래스에 상속받고 싶은 경우

함수의 인터페이스 및 구현을 모두 상속받고 그 상속받은 구현이 오버라이드가 가능하게 만들었으면 하는 경우

인터페이스와 구현을 상속받되 어떤 것도 오버라이드할 수 없도록 막고 싶은 경우

---

기하학적 도형을 나타내는 클래스

```c++
class Shape {
public:
    virtual void draw() const = 0;
    virtual void error(const std::string& msg);
    int objectID() const;
    ...
};
class Rectangle: public Shape { ... };
class Ellipse: public Shape { ... };
```

draw가 순수 가상 함수 → Shape는 추상 클래스

Shape 클래스의 인스턴스를 만들려고 하면 안 되고, 이 클래스의 파생 클래스만 인스턴스화가 가능함

Shape가 이 클래스로부터 파생된 클래스에 대해 미치는 영향은 큼

- 멤버 함수 인터페이스는 항상 상속되게 되어 있기 때문. public 상속의 의미는 is-a. 어떤 클래스에서 동작하는 함수는 그 클래스의 파생 클래스에서도 동작해야 함

Shape 클래스에는 세 개의 함수가 선언되어 있음

draw 함수는 암시적인 표시장치에 현재의 객체를 그림

error 함수는 다른 멤버 함수들이 호출하는 함수로, 이들이 에러를 보고할 필요가 있을 때 사용됨

objectID 함수는 주어진 객체에 붙는 유일한 정수 식별자를 반환함

세 함수는 선언된 방법도 각기 다름

draw는 순수 가상 함수

error는 단순 가상 함수

objectID는 비가상 함수

---

**순수 가상 함수**

어떤 순수 가상 함수를 물려받은 구체 클래스가 해당 순수 가상 함수를 다시 선언해야 함

순수 가상 함수는 전형적으로 추상 클래스 안에서 정의를 갖지 않음

- 순수 가상 함수를 선언하는 목적은 파생 클래스에게 함수의 인터페이스만을 물려주라는 것

---

순수 가상 함수에도 정의를 제공할 수 있음

단, 구현이 붙은 순수 가상 함수를 호출하려면 반드시 클래스 이름을 한정자로 붙여 주어야 함(추상 클래스로는 인스턴스를 만들 수 없으므로)

단순 가상 함수에 대한 기본 구현을 안전하게 제공하는 메커니즘으로도 활용할 수 있음

---

**단순(비순수) 가상 함수**

파생 클래스로 하여금 함수의 인터페이스를 상속하게 한다는 점은 순수 가상 함수와 똑같지만, 파생 클래스 쪽에서 오버라이드할 수 있는 함수 구현부도 제공한다는 점이 다름

- 단순 가상 함수를 선언하는 목적은 파생 클래스로 하여금 함수의 인터페이스뿐만 아니라 그 함수의 기본 구현도 물려받게 하자는 것

---

단순 가상 함수에서 함수 인터페이스와 기본 구현을 한꺼번에 지정하도록 내버려두는 것은 위험할 수도 있음

비행기 A 모델과 B 모델은 비행 방식이 똑같음

```c++
class Airport { ... };

class Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void Airplane::fly(const Airport& destination) {
    // 주어진 목적지로 비행기를 날려 보내는 기본 동작 원리를 가진 코드
}

class ModelA: public Airplane { ... };
class ModelB: public Airplane { ... };
```

Airplane::fly 함수는 가상 함수 → 모든 비행기는 fly 함수를 지원해야 함. 모델이 다른 비행기는 원칙상 fly 함수에 대한 구현을 저마다 다르게 요구할 수 있음

ModelA, ModelB가 물려받을 수 있도록 기본적인 비행 원리를 Airplane::fly 함수의 본문으로 제공

클래스 사이의 공통 사항으로 둘 수 있는 특징이 명확해짐

코드가 중복되지 않음

이후의 기능 개선의 통로가 열려 있음

장기적인 유지보수가 쉬워짐

---

C 모델은 A 모델, B 모델과 비행 방식이 완전히 다름

fly 함수 재정의를 잊어버림

```c++
class ModelC: public Airplane {
    ...  // fly 함수가 선언되지 않음
};
```

ModelC 객체가 마치 ModelA, ModelB라도 된 것처럼 날려보내게 됨

ModelC 클래스는 기본 동작을 원한다고 명시적으로 밝히지 않았는데도 이 동작을 물려받는 데 아무런 걸림돌이 없음

기본 동작을 파생 클래스에서 요구하지 않으면 주지 않는 방법

가상 함수의 인터페이스와 그 가상 함수의 기본 구현을 잇는 연결 관계 끊기

```c++
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
protected:
    void defaultFly(const Airport& destination);
};

void Airplane::defaultFly(const Airport& destination) {
    // 주어진 목적지로 비행기를 날려 보내는 기본 동작 원리를 가진 코드
}
```

Airplane::fly 함수가 순수 가상 함수로 바뀜 → fly 함수의 인터페이스를 제공하는 역할

기본 동작을 쓰고 싶은 클래스에서는 fly 함수의 본문 안에서 defaultFly 함수를 인라인 호출하기만 하면 됨

```c++
class ModelA: public Airplane {
public:
    virtual void fly(const Airport& destination) { defaultFly(destination); }
    ...
};
```

ModelC 클래스가 자신과 맞지 않는 기본 구현을 우연찮게 물려받을 가능성은 없어짐

fly 함수가 Airplane 클래스의 순수 가상 함수 → ModelC 자신만의 버전을 스스로 제공해야 함

Airplane::defaultFly 함수는 protected 멤버 → Airplane 및 그 클래스의 파생 클래스만 내부적으로 사용하는 구현 세부사항이기 때문

Airplane::defaultFly 함수는 비가상 함수 → 파생 클래스 쪽에서 이 함수를 재정의해선 안 되기 때문

---

순수 가상 함수가 구체 파생 클래스에서 재선언되어야 한다는 사실을 활용하되, 자체적으로 순수 가상 함수의 구현을 구비해 두는 것

```c++
class Airplane {
public:
    virtual void fly(const Airport& destination) = 0;
    ...
};

void Airplane::fly(const Airport& destination) {
    // 주어진 목적지로 비행기를 날려 보내는 기본 코드
}

class ModelA: public Airplane {
public:
    virtual void fly(const Airport& destination) { Airplane::fly(destination); }
    ...
};

class ModelB: public Airplane {
public:
    virtual void fly(const Airport& destination) { Airplane::fly(destination); }
    ...
};

class ModelC: public Airplane {
public:
    virtual void fly(const Airport& destination);
    ...
};

void ModelC::fly(const Airport& destination) {
    // 주어진 목적지로 ModelC 비행기를 날려 보내는 코드
}
```

fly 함수가 선언부 및 정의부의 두 쪽으로 나뉨

선언부는 이 함수의 인터페이스를 지정하고, 정의부는 이 함수의 기본 동작을 지정함

protected 영역에 있었던 코드가 public 영역에 있게 됨

---

**비가상 함수**

objectID

```c++
class Shape {
public:
    int objectID() const;
    ...
};
```

비가상 멤버 함수는 클래스 파생에 상관없이 변하지 않는 동작을 지정하는 데 쓰임

- 비가상 함수를 선언하는 목적은 파생 클래스가 함수 인터페이스와 더불어 그 함수의 필수적인 구현(mandatory implementation)을 물려받게 하는 것

---

순수 가상 함수, 단순 가상 함수, 비가상 함수의 선언문이 가진 이런저런 차이점 덕택에, 파생 클래스가 물려받았으면 하는 것들을 정밀하게 지정 가능

---

클래스에서 가장 흔히 발견되는 결정적인 실수 두 가지

---

첫 번째 실수는 모든 멤버 함수를 비가상 함수로 선언하는 것

파생 클래스를 만들더라도 기본 클래스의 동작을 특별하게 만들 만한 여지가 없어짐

특히 비가상 소멸자가 문제가 될 수 있음 → Item 7 참고

클래스 파생을 처음부터 염두에 두지 않은 클래스를 설계하는 경우는 비가상 함수들만 모아두는 게 맞음

80-20 법칙 → Item 30 참고

어지간한 프로그램에서는 전체 진행 시간의 80%가 소모되는 부분이 전체 코드의 20%밖에 되지 않는다는 법칙

정말 특수한 상황이 아니면 함수 호출 중 80%를 가상 함수로 두더라도 프로그램의 전체 수행 성능에는 가장 약하게 느낄 수 있을 만큼의 손실도 생기지 않는다는 뜻

가상 함수의 비용을 무느냐 안 무느냐에 따라 진짜 큰 차이를 만들 수 있는 20%의 코드에 더 집중

---

두 번째 실수는 모든 멤버 함수를 가상 함수로 선언하는 것

맞을 경우도 없는 것은 아님

파생 클래스에서 재정의가 안 되어야 하는 함수가 있으면 반드시 비가상 함수로 만들기

---

**인터페이스 상속은 구현 상속과 다름. public 상속에서, 파생 클래스는 항상 기본 클래스의 인터페이스를 모두 물려받음**

**순수 가상 함수는 인터페이스 상속만을 허용함**

**단순(비순수) 가상 함수는 인터페이스 상속과 더불어 기본 구현의 상속도 가능하도록 지정함**

**비가상 함수는 인터페이스 상속과 더불어 필수 구현의 상속도 가하도록 지정함**

