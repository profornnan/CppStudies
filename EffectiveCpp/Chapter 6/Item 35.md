# Item 35: 가상 함수 대신 쓸 것들도 생각해 두는 자세를 시시때때로 길러 두자

캐릭터 클래스 설계

healthValue 함수는 캐릭터의 체력이 얼마나 남았는지를 나타내는 정수 값을 반환함

체력 계산은 캐릭터마다 다를 것이므로, 이 함수를 가상 함수로 선언하는 것이 확실한 설계일 것 같음

```c++
class GameCharacter {
public:
    virtual int healthValue() const;  // 파생 클래스는 이 함수를 재정의할 수 있음
    ...
};
```

healthValue 함수가 순수 가상 함수로 선언되지 않음 → 체력 계산 기본 알고리즘이 제공됨 → Item 34 참고

### 비가상 인터페이스 관용구를 통한 템플릿 메서드 패턴

가상 함수 은폐론 → 가상 함수는 반드시 private 멤버로 두어야 함

healthValue를 public 멤버 함수로 그대로 두되 비가상 함수로 선언하고, 내부적으로는 실제 동작을 맡은 private 가상 함수를 호출하는 식으로 만드는 것

```c++
class GameCharacter {
public:
    int healthValue() const {  // 파생 클래스는 이 함수를 재정의할 수 없음
        ...  // 사전 동작
        int retVal = doHealthValue();  // 실제 동작
        ...  // 사후 동작
        return retVal;
    }
    ...
private:
    virtual int doHealthValue() const {  // 파생 클래스는 이 함수를 재정의할 수 있음
        ...  // 체력 계산 기본 알고리즘
    }
};
```

멤버 함수의 본문이 클래스 정의 안에 들어가 있음 → 암시적으로 인라인 함수로 선언됨 → Item 30 참고

사용자로 하여금 public 비가상 멤버 함수를 통해 private 가상 함수를 간접적으로 호출하게 만드는 방법

**비가상 함수 인터페이스(non-virtual interface: NVI) 관용구**

템플릿 메서드(Template Method)(C++ 템플릿하고는 아무 관계가 없는 패턴 이름)라 불리는 고전 디자인 패턴을 C++ 식으로 구현한 것

이 관용구에 쓰이는 비가상 함수를 가상 함수의 랩퍼(wrapper)라고 함

---

NVI 관용구의 이점

사전 동작, 사후 동작

가상 함수가 호출되기 전에 어떤 상태를 구성하고 가상 함수가 호출된 후에 그 상태를 없애는 작업이 랩퍼를 통해 공간적으로 보장된다는 뜻

뮤텍스 잠금, 로그 정보 생성, 클래스의 불변속성과 함수의 사전조건이 만족되었나를 검증

뮤텍스 잠금 해제, 함수의 사후조건을 점검하고 클래스의 불변속성을 재검증

---

어떤 기능을 어떻게 구현할지를 조정하는 권한은 파생 클래스가 갖게 되지만, 함수를 언제 호출할지를 결정하는 것은 기본 클래스만의 고유 권한임

상속받은 private 가상 함수를 파생 클래스가 재정의할 수 있음

### 함수 포인터로 구현한 전략 패턴

NVI 관용구는 public 가상 함수를 대신할 수 있지만, 체력치를 계산하는 데 가상 함수를 사용하는 것은 여전함

체력치 계산이 어떤 캐릭터의 일부일 필요가 없음

각 캐릭터의 생성자에 체력치 계산용 함수의 포인터를 넘기게 만들고, 이 함수를 호출해서 실제 계산 수행

```c++
class GameCharacter;

int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
public:
    typedef int (*HealthCalcFunc)(const GameCharacter&);
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc): healthFunc(hcf) {}
    int healthValue() const { return healthFunc(*this); }
    ...
private:
    HealthCalcFunc healthFunc;
};
```

디자인 패턴인 전략(Strategy) 패턴의 단순한 응용 예

- 같은 캐릭터 타입으로부터 만들어진 객체(인스턴스)들도 체력치 계산 함수를 각각 다르게 가질 수 있음
- 게임이 실행되는 도중에 특정 캐릭터에 대한 체력치 계산 함수를 바꿀 수 있음

---

체력치 계산 함수가 GameCharacter 클래스 계통의 멤버 함수가 아니라는 점은, 체력치가 계산되는 대상 객체의 비공개 데이터는 이 함수로 접근할 수 없다는 뜻

정확한 체력치 계산을 위해서 public 멤버가 아닌 정보를 써야 할 경우 문제 발생

public 영역에 없는 부분을 비멤버 함수도 접근할 수 있게 하려면 그 클래스의 캡슐화를 약화시키는 방법밖에는 없다는 것이 일반적인 법칙

ex) 비멤버 함수를 프렌드로 선언, 세부 구현사항에 대한 접근자 함수를 public 멤버로 제공

함수 포인터를 통해 얻는 이점들이 클래스의 캡슐화를 떨어뜨리면서 얻는 불이익을 채워 줄지 아닐지 판단

### tr1::function으로 구현한 전략 패턴

tr1::function 계열의 객체는 함수호출성 개체(callable entity)(함수 포인터, 함수 객체 혹은 멤버 함수 포인터)를 가질 수 있고, 이들 개체는 주어진 시점에서 예상되는 시그너처와 호환되는 시그너처를 갖고 있음

```c++
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
public:
    typedef std::tr1::function<int (const GameCharacter&)> HealthCalcFunc;
    explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc): healthFunc(hcf) {}
    int healthValue() const { return healthFunc(*this); }
    ...
private:
    HealthCalcFunc healthFunc;
};
```

HealthCalcFunc는 함수호출성 개체로서, GameCharacter와 호환되는 어떤 것이든 넘겨받아서 호출될 수 있으며 int와 호환되는 모든 타입의 객체를 반환함

HealthCalcFunc는 tr1::function 템플릿을 인스턴스화한 것에 대한 typedef 타입 → 일반화된 함수 포인터 타입처럼 동작

```c++
std::tr1::function<int (const GameCharacter&)>
```

const GameCharacter에 대한 참조자를 받고 int를 반환하는 함수

이렇게 정의된 tr1::function 타입으로 만들어진 객체는 앞으로 대상 시그너처와 호환되는 함수호출성 개체를 어떤 것도 가질 수 있음

호환된다(compatible)라는 말은, 함수호출성 개체의 매개변수 타입이 const GameCharacter&이거나 const GameCharacter&으로 암시적 변환이 가능한 타입이며, 반환 타입도 암시적으로 int로 변환될 수 있다는 뜻

GameCharacter가 좀더 일반화된 함수 포인터를 물게 됨

```c++
short calcHealth(const GameCharacter&);

struct HealthCalculator {
    int operator()(const GameCharacter&) const { ... }
};

class GameLevel {
public:
    float health(const GameCharacter&) const;
    ...
};

class EvilBadGuy: public GameCharacter { ... };
class EyeCandyCharacter: public GameCharacter { ... };

EvilBadGuy ebg1(calcHealth);  // 체력치 계산을 위한 함수를 사용하는 캐릭터
EyeCandyCharacter ecc1(HealthCalculator());  // 체력치 계산을 위한 함수 객체를 사용하는 캐릭터
GameLevel currentLevel;
...
EvilBadGuy ebg2(std::tr1::bind(&GameLevel::health, currentLevel, _1));  // 체력치 계산을 위한 멤버 함수를 사용하는 캐릭터
```

ebg2 정의문이 말하는 바는, ebg2의 체력치를 계산하기 위해 GameLevel 클래스의 health 멤버 함수를 써야 한다는 것

GameLevel::health 함수는 매개변수 하나를 받는 것으로 선언되어 있지만, 실제로는 두 개를 받음

GameLevel 객체 하나를 암시적으로 받아들임 → this 포인터가 가리키는 것

하지만 GameCharacter 객체에 쓰는 체력치 계산 함수가 받는 매개변수는 체력치가 계산되는 GameCharacter 객체 하나뿐임

매개변수 두 개(GameCharacter, GameLevel)를 받는 함수를 매개변수 한 개(GameCharacter)만 받는 함수로 바꿔야 함

GameLevel::health 함수가 호출될 때마다 currentLevel이 사용되도록 tr1::bind로 묶어준 것

ebg2의 체력치 계산 함수는 항상 currentLevel만을 GameLevel 객체로 쓴다고 지정한 것

"_1"은 ebg2에 대해 currentLevel과 묶인 GameLevel::health 함수를 호출할 때 넘기는 첫 번째 자리의 매개변수를 뜻함

함수 포인터 대신 tr1::function을 사용함으로써, 체력치를 계산할 때 시그너처가 호환되는 함수호출성 개체는 어떤 것도 원하는 대로 구사할 수 있도록 융통성을 열어줌

### 고전적인 전략 패턴

체력치 계산 함수를 나타내는 클래스 계통을 아예 따로 만들고, 실제 체력치 계산 함수는 이 클래스 계통의 가상 멤버 함수로 만드는 것

GameCharacter가 상속 계통의 최상위 클래스이고 EvilBadGuy 및 EyeCandyCharacter는 여기서 갈라져 나온 파생 클래스

HealthCalcFunc는 SlowHealthLoser 및 FastHealthLoser 등을 파생 클래스로 거느린 최상위 클래스

GameCharacter 타입을 따르는 모든 객체는 HealthCalcFunc 타입의 객체에 대한 포인터를 포함함

```c++
class GameCharacter;

class HealthCalcFunc {
public:
    ...
    virtual int calc(const GameCharacter& gc) const { ... }
    ...
};

HealthCalcFunc defaultHealthCalc;

class GameCharacter {
public:
    explicit GameCharacter(HealthCalcFunc *phcf = &defaultHealthCalc): pHealthCalc(phcf) {}
    int healthValue() const { return pHealthCalc->calc(*this); }
    ...
private:
    HealthCalcFunc *pHealthCalc;
};
```

HealthCalcFunc 클래스 계통에 파생 클래스를 추가함으로써 기존의 체력치 계산 알고리즘을 조정/개조할 수 있는 가능성을 열어 놓음

### 지금까지 공부한 것들에 대한 요약

어떤 문제를 해결하기 위한 설계를 찾을 때 가상 함수를 대신하는 방법들도 고려해 보자

- **비가상 인터페이스 관용구**(NVI 관용구) 사용 : 공개되지 않은 가상 함수를 비가상 public 멤버 함수로 감싸서 호출하는, 템플릿 메서드 패턴의 한 형태
- 가상 함수를 **함수 포인터 데이터 멤버**로 대체 : 전략 패턴의 핵심만을 보여주는 형태
- 가상 함수를 **tr1::function 데이터 멤버**로 대체하여, 호환되는 시그너처를 가진 함수호출성 개체를 사용할 수 있도록 만들기 : 전략 패턴의 한 형태
- 한쪽 클래스 계통에 속해 있는 가상 함수를 **다른 쪽 계통에 속해 있는 가상 함수**로 대체 : 전략 패턴의 전통적인 구현 형태

---

**가상 함수 대신에 쓸 수 있는 다른 방법으로 NVI 관용구 및 전략 패턴을 들 수 있음. NVI 관용구는 그 자체가 템플릿 메서드 패턴의 한 예**

**객체에 필요한 기능을 멤버 함수로부터 클래스 외부의 비멤버 함수로 옮기면, 그 비멤버 함수는 그 클래스의 public 멤버가 아닌 것들을 접근할 수 없다는 단점이 생김**

**tr1::function 객체는 일반화된 함수 포인터처럼 동작함. 이 객체는 주어진 대상 시그너처와 호환되는 모든 함수호출성 개체를 지원함**

