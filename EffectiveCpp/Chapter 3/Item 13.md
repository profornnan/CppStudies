# Item 13: 자원 관리에는 객체가 그만!

투자 모델링 클래스 라이브러리로 작업한다고 가정

최상위 클래스는 Investment

```c++
class Investment { ... };
```

Investment에서 파생된 클래스의 객체를 사용자가 얻어내는 용도로 팩토리 함수만을 쓰도록 만들어져 있음

```c++
Investment* createInvestment();
```

createInvestment 함수를 통해 얻어낸 객체를 사용할 일이 없을 때 그 객체를 삭제해야 하는 쪽은 이 함수의 호출자

```c++
void f() {
    Investment *pInv = createInvestment();
    ...
    delete pInv;
}
```

투자 객체 삭제에 실패할 수 있는 경우 많음

... 부분에 return문이 들어있는 경우

createInvestment 호출문과 delete가 하나의 루프 안에 들어 있고 continue 혹은 goto문에 의해 루프에서 빠져 나오는 경우

... 안의 문장에서 예외가 발생한 경우

createInvestment 함수로 얻어낸 자원이 항상 해제되도록 만들 방법은, 자원을 객체에 넣고 그 자원 해제를 소멸자가 맡도록 하며, 그 소멸자는 실행 제어가 f를 떠날 때 호출되도록 만드는 것

블록 혹은 함수로부터 실행 제어가 빠져나올 때 자원 해제

---

auto_ptr : 스마트 포인터

포인터와 비슷하게 동작하는 객체로, 가리키고 있는 대상에 대해 소멸자가 자동으로 delete 호출

```c++
void f() {
    std::auto_ptr<Investment> pInv(createInvestment());
    ...
}
```

auto_ptr의 소멸자를 통해 pInv 삭제

---

### 자원 관리에 객체를 사용하는 방법의 두 가지 특징

#### 1. 자원을 획득한 후에 자원 관리 객체에게 넘김

자원 획득 즉 초기화(Resource Acquisition Is Initialization: RAII)

자원 획득과 자원 관리 객체의 초기화가 한 문장에서 이뤄짐

#### 2. 자원 관리 객체는 자신의 소멸자를 사용해서 자원이 확실하게 해제되도록 함

소멸자는 어떤 객체가 소멸될 때 자동적으로 호출되기 때문에 자원 해제가 제대로 이뤄지게 됨

---

auto_ptr은 자신이 소멸될 때 자신이 가리키고 있는 대상을 자동으로 delete함

→ 어떤 객체를 가리키는 auto_ptr의 개수가 둘 이상이면 절대로 안 됨

auto_ptr 객체를 복사하면 원본 객체는 null로 만든다.

복사하는 객체만이 그 자원의 유일한 소유권 가짐

STL 컨테이너는 원소들이 정상적인 복사 동작을 가져야 하기 때문에 auto_ptr은 이들의 원소로 허용되지 않음

---

auto_ptr을 쓸 수 없는 경우 참조 카운팅 방식 스마트 포인터(reference-counting smart pointer: RCSP)가 괜찮은 대안임

RCSP는 특정한 어떤 자원을 참조하는 외부 객체의 개수를 유지하고 있다가 그 개수가 0이 되면 해당 자원을 자동으로 삭제하는 스마트 포인터

TR1에서 제공되는 tr1::shared_ptr이 대표적인 RCSP

```c++
void f() {
    ...
    std::tr1::shared_ptr<Investment> pInv1(createInvestment());
    std::tr1::shared_ptr<Investment> pInv2(pInv1);
    pInv1 = pInv2;
}
```

복사 동작이 예상대로 이뤄짐 → auto_ptr을 쓸 수 없는 STL 컨테이너 등의 환경에 적합

auto_ptr 및 tr1::shared_ptr은 소멸자 내부에서 delete[] 연산자가 아닌 delete 연산자를 사용함

```c++
std::auto_ptr<std::string> aps(new std::string[10]);
```

잘못된 delete가 사용됨

C++ 표준 라이브러리에서는 동적 할당된 배열을 위해 준비된 스마트 포인터가 제공되지 않음

동적으로 할당된 배열은 vector 및 string으로 거의 대체할 수 있기 때문

---

**자원 관리에 객체를 쓰자**

자원 해제 일일이 하다 보면 실수 발생 가능

자원 관리 클래스 활용(auto_ptr, tr1::shared_ptr)

자원 관리 클래스 직접 구현

---

예제에서 createInvestment 함수의 반환 타입이 포인터

반환된 포인터에 대한 delete 호출 잊어버리기 쉬움

인터페이스 수정 → Item 18 참고

---

**자원 누출을 막기 위해, 생성자 안에서 자원을 획득하고 소멸자에서 그것을 해제하는 RAII 객체 사용**

**일반적으로 널리 쓰이는 RAII 클래스는 tr1::shared_ptr과 auto_ptr**

tr1::shared_ptr은 복사 시 동작 직관적

auto_ptr은 원본 객체를 null로 만듦

