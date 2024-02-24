# Item 14: 자원 관리 클래스의 복사 동작에 대해 진지하게 고찰하자

자원 획득 즉 초기화(RAII)

힙 기반 자원 → auto_ptr, tr1::shared_ptr

힙에 생기지 않는 자원을 스마트 포인터로 처리하는 것은 일반적으로 맞지 않음

자원 관리 클래스 직접 구현

---

Mutex 타입의 뮤텍스 객체를 조작하는 C API 사용 가정

```c++
void lock(Mutex *pm);
void unlock(Mutex *pm);
```

뮤텍스 잠금 관리 클래스 구현

뮤텍스 잠금 잊지 않고 풀어주기

RAII 법칙 따라 구성 → 생성 시 자원 획득, 소멸 시 자원 해제

```c++
class Lock {
public:
    explicit Lock(Mutex *pm): mutexPtr(pm) { lock(mutexPtr); }  // 자원 획득
    ~Lock() { unlock(mutexPtr); }  // 자원 해제

private:
    Mutex *mutexPtr;
};
```

RAII 방식에 맞춰서 사용

```c++
Mutex m;  // 뮤텍스 정의
...
{  // 임계 영역 정하기 위한 블록 생성
    Lock m1(&m);  // 뮤텍스 잠금
    ...  // 임계 영역에서 할 연산 수행
}  // 뮤텍스 잠금 자동으로 해제
```

Lock 객체가 복사된다면?

### RAII 객체가 복사될 때 어떤 동작이 이뤄져야 할까?

#### 복사 금지

복사하면 안 되는 RAII 클래스에 대해서는 반드시 복사를 막아야 함

복사 연산을 private 멤버로 만들기 → Item 6 참고

#### 관리하고 있는 자원에 대해 참조 카운팅 수행

자원을 사용하고 있는 마지막 객체가 소멸될 때까지 자원 해제를 안 하는게 바람직한 경우, 해당 자원을 참조하는 객체의 개수에 대한 카운트를 증가시키는 식으로 RAII 객체의 복사 동작을 만들어야 함

tr1::shared_ptr이 사용하는 방식

tr1::shared_ptr은 삭제자(deleter) 지정을 허용함 → 생성자의 두 번째 매개변수

삭제자 : tr1::shared_ptr이 유지하는 참조 카운트가 0이 되었을 때 호출되는 함수 혹은 객체

auto_ptr에는 해당 기능이 없음

```c++
class Lock {
public:
    explicit Lock(Mutex *pm): mutexPtr(pm, unlock) {
        lock(mutexPtr.get());
    }

private:
    std::tr1::shared_ptr<Mutex> mutexPtr;
}
```

shared_ptr 초기화 → 가리킬 포인터로 Mutex 객체의 포인터 사용, 삭제자로 unlock 함수 사용

get → Item 15 참고

소멸자 선언 X

클래스의 소멸자는 비정적 데이터 멤버의 소멸자를 자동으로 호출

mutexPtr은 비정적 데이터 멤버

mutexPtr의 소멸자는 뮤텍스의 참조 카운트가 0이 될 때 tr1::shared_ptr의 삭제자 unlock을 자동으로 호출

