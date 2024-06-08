# Item 29: 예외 안전성이 확보되는 그날 위해 싸우고 또 싸우자!

GUI 메뉴 클래스

스레딩 환경에서 동작 → 병행성 제어를 위해 뮤텍스(mutex) 사용

```c++
class PrettyMenu {
public:
    ...
    void changeBackground(std::istream& imgSrc);
    ...
private:
    Mutex mutex;
    Image *bgImage;
    int imageChanges;
};
```

changeBackground 함수

```c++
void PrettyMenu::changeBackground(std::istream& imgSrc) {
    lock(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
    unlock(&mutex);
}
```

예외 안전성 측면에서 안 좋은 함수

### 예외 안전성을 가진 함수라면 예외가 발생할 때 이렇게 동작해야 함

#### 자원이 새도록 만들지 않기

"new Image(imgSrc)" 표현식에서 예외를 던지면 unlock 함수가 실행되지 않게 되어 뮤텍스가 계속 잡힌 상태로 남아있게 되어 자원이 샘

#### 자료구조가 더럽혀지는 것을 허용하지 않기

"new Image(imgSrc)"가 예외를 던지면 bgImage가 가리키는 객체는 이미 삭제된 후

새 그림이 제대로 깔린 게 아닌데도 imageChanges 변수는 이미 증가된 상태

---

자원 누출 문제

객체를 써서 자원 관리를 전담케 하는 방법 → Item 13

뮤텍스를 적절한 시점에 해제하는 방법을 구현한 Lock 클래스 → Item 14 참고

```c++
void PrettyMenu::changeBackground(std::istream& imgSrc) {
    Lock ml(&mutex);
    delete bgImage;
    ++imageChanges;
    bgImage = new Image(imgSrc);
}
```

---

자료구조 오염 문제

### 예외 안전성을 갖춘 함수는 아래의 세 가지 보장(guarantee) 중 하나를 제공함

#### 기본적인 보장(basic guarantee)

함수 동작 중 예외 발생 시, 실행 중인 프로그램에 관련된 모든 것들을 유효한 상태로 유지하겠다는 보장

어떤 객체나 자료구조도 더럽혀지지 않으며, 모든 객체의 상태는 내부적으로 일관성을 유지하고 있음(즉, 모든 클래스 불변속성이 만족된 상태)

하지만 프로그램의 상태가 정확히 어떠한지는 예측이 안 될 수도 있음

예외 발생 시 상태가 유효하기만 하면 어떤 상태도 될 수 있음

#### 강력한 보장(strong guarantee)

함수 동작 중 예외 발생 시, 프로그램의 상태를 절대로 변경하지 않겠다는 보장

원자적인(atomic) 동작

호출이 성공하면 마무리까지 완벽하게 성공하고, 호출이 실패하면 함수 호출이 없었던 것처럼 프로그램의 상태가 되돌아감

기본 보장을 제공하는 함수보다 더 쓰기 쉬움

예측할 수 있는 프로그램의 상태가 두 개밖에 안 됨

#### 예외불가 보장(nothrow guarantee)

예외를 절대로 던지지 않겠다는 보장

약속한 동작은 언제나 끝까지 완수하는 함수

기본제공 타입에 대한 모든 연산은 예외를 던지지 않게 되어 있음(즉, 예외불가 보장이 제공됨)

예외에 안전한 코드를 만들기 위한 가장 기본적이며 핵심적인 요소

함수가 어떤 특성을 갖느냐 하는 부분은 선언이 아니라 구현이 결정함

---

아무 보장도 제공하지 않으면 예외에 안전한 함수가 아님

어떤 보장을 제공할 것인지 선택

현실적으로는 대부분의 함수에 있어서 기본적인 보장과 강력한 보장 중 하나를 고르게 됨

실용성이 있는 강력한 보장

---

강력한 보장을 거의 제공하도록 changeBackground 함수 수정

PrettyMenu의 bgImage 데이터 멤버의 타입을 기본제공 포인터 타입인 Image*에서 자원관리 전담용 포인터(Item 13 참고)로 변경

changeBackground 함수 내의 문장을 재배치해서 배경그림이 진짜로 바뀌기 전에는 imageChanges를 증가시키지 않도록 함

```c++
class PrettyMenu {
    ...
    std::tr1::shared_ptr<Image> bgImage;
    ...
};

void PrettyMenu::changeBackground(std::istream& imgSrc) {
    Lock ml(&mutex);
    bgImage.reset(new Image(imgSrc));  // bgImage의 내부 포인터를 "new Image" 표현식의 실행 결과로 바꿔치기함
    ++imageChanges;
}
```

배경그림(Image 객체)이 스마트 포인터의 손에서 관리되고 있기 때문에 프로그래머가 직접 삭제할 필요가 없음

---

매개변수 imgSrc가 옥의 티

Image 클래스의 생성자가 실행되다가 예외를 일으킬 때, 그 시점에 입력 스트림의 읽기 표시자가 이동한 채로 남아 있을 가능성이 있음

엄밀히 말하면 changeBackground가 제공하는 예외 안전성 보장은 기본적인 보장임

매개변수 타입으로 istream을 쓰지 말고, 배경그림 파일의 이름을 나타내는 타입으로 변경

---

복사-후-맞바꾸기(copy-and-swap)

예외에 속수무책인 함수를 탈바꿈시켜 강력한 예외 안전성 보장을 제공하는 함수로 거듭나게 만드는 일반적인 설계 전략

어떤 객체를 수정하고 싶으면 그 객체의 사본을 하나 만들어 놓고 그 사본을 수정하는 것

수정 동작 중에 실행되는 연산에서 예외가 던져지더라도 원본 객체는 바뀌지 않음

필요한 동작이 전부 성공적으로 완료되고 나면 수정된 객체를 원본 객체와 맞바꾸는데, 이 작업을 예외를 던지지 않는 연산 내부에서 수행함

이 전략은 대개 진짜 객체의 모든 데이터를 별도의 구현 객체에 넣어두고, 그 구현 객체를 가리키는 포인터를 진짜 객체가 물고 있게 하는 식으로 구현함

pimpl 관용구 → Item 31 참고

```c++
struct PMImpl {
    std::tr1::shared_ptr<Image> bgImage;
    int imageChanges;
};

class PrettyMenu {
    ...
private:
    Mutex mutex;
    std::tr1::shared_ptr<PMImpl> pImpl;
};

void PrettyMenu::changeBackground(std::istream& imgSrc) {
    using std::swap;
    Lock ml(&mutex);
    std::tr1::shared_ptr<PMImpl> pNew(new PMImpl(*pImpl));
    pNew->bgImage.reset(new Image(imgSrc));
    ++pNew->imageChanges;
    swap(pImpl, pNew);
}
```

PMImpl이 클래스가 아니라 구조체로 만들어져 있음 → PrettyMenu 클래스에서 pImpl이 private 멤버로 되어 있어서 구현 객체의 데이터가 바로 캡슐화 됨

클래스로 만들어도 괜찮음

---

복사-후-맞바꾸기 전략은 객체의 상태를 all-or-nothing 방식으로 유지하려는 경우에 좋지만, 함수 전체가 강력한 예외 안전성을 갖도록 보장하지는 않음

changeBackground 함수의 전체 흐름을 추상화해 놓은 someFunc()

```c++
void someFunc() {
    ...  // 이 함수의 현재 상태에 대해 사본 생성
    f1();
    f2();
    ...  // 변경된 상태를 바꾸어 넣음
}
```

f1 혹은 f2에서 보장하는 예외 안전성이 강력하지 못하면, 위의 구조로는 someFunc 함수 역시 강력한 예외 안전성을 보장하기 힘들어짐

f1 및 f2 모두가 강력한 예외 안전성을 보장한다고 해도 사실 별로 나아지는 것은 없음 → f1이 끝까지 실행되고 나면 프로그램 상태는 f1에 의해 어떻게든 변해 있을 것임

함수의 부수효과(side effect)

자기 자신에만 국한된 것들의 상태를 바꾸며 동작하는 함수의 경우에는 강력한 보장을 제공하기가 비교적 수월하지만, 비지역 데이터에 대해 부수효과를 주는 함수는 이렇게 하기가 무척 까다로움

---

효율 문제도 무시할 수 없음

수정하고 싶은 객체를 복사해 둘 공간과 복사에 걸리는 시간을 감수해야 함

---

실용성이 확보될 때만 강력한 보장 제공

대다수의 함수에 있어서 기본적인 보장이 우선

예외 안전성 보장을 아예 제공하지 않는 함수를 만들지 말자

---

예외 안전성이 없는 함수가 한 개라도 쓰이고 있으면 그 시스템은 전부가 예외에 안전하지 않은 시스템임

C++로 작성된 상당수의 재래식 코드가 예외 안전성 자체를 고려하지 않고 만들어진 것이 사실임

어떻게 하면 예외에 안전한 코드를 만들지 진지하게 고민하는 버릇을 들이자

자원 관리 필요 시 자원 관리용 객체 사용

예외 안전성 보장 세 가지 중 함수에 실용적으로 제공할 수 있는 보장이 어떤 것인지 결정

---

**예외 안전성을 갖춘 함수는 실행 중 예외가 발생되더라도 자원을 누출시키지 않으며 자료구조를 더럽힌 채로 내버려 두지 않음. 이런 함수들이 제공할 수 있는 예외 안전성 보장은 기본적인 보장, 강력한 보장, 예외 금지 보장이 있음**

**강력한 예외 안전성 보장은 복사-후-맞바꾸기 방법을 써서 구현할 수 있지만, 모든 함수에 대해 강력한 보장이 실용적인 것은 아님**

**어떤 함수가 제공하는 예외 안전성 보장의 강도는, 그 함수가 내부적으로 호출하는 함수들이 제공하는 가장 약한 보장을 넘지 않음**

