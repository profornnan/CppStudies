# Item 44: 매개변수에 독립적인 코드는 템플릿으로부터 분리시키자

아무 생각 없이 템플릿을 사용하면 **코드 비대화(code bloat)**가 초래될 수 있음

똑같은 내용의 코드와 데이터가 여러 벌로 중복되어 이진 파일로 구워진다는 뜻

---

**공통성 및 가변성 분석(commonality and variability analysis)**

두 함수를 분석해서 공통적인 부분과 다른 부분을 찾은 후에 공통 부분은 새로운 함수에 옮기고 다른 부분은 원래의 함수에 남겨두는 것

클래스의 경우도 비슷함

공통 부분을 별도의 새로운 클래스에 옮긴 후, 클래스 상속 혹은 객체 합성을 사용해서 원래의 클래스들이 공통 부분을 공유하도록 함

템플릿을 작성할 경우에도 똑같은 분석을 하고 똑같은 방법으로 코드 중복을 막으면 됨

템플릿이 아닌 코드에서는 코드 중복이 명시적

템플릿 코드에서는 코드 중복이 암시적

---

고정 크기의 정방행렬을 나타내는 클래스 템플릿

```c++
template<typename T, std::size_t n>
class SquareMatrix {
public:
    ...
    void invert();
};
```

T 타입의 객체를 원소로 하는 n행 n열의 행렬을 나타내는 템플릿

size_t는 비타입 매개변수(non-type parameter)

```c++
SquareMatrix<double, 5> sm1;
...
sm1.invert();

SquareMatrix<double, 10> sm2;
...
sm2.invert();
```

두 개의 invert 사본이 만들어짐

행과 열의 크기를 나타내는 상수만 빼면 두 함수는 완전히 똑같음

코드 비대화

n값을 매개변수로 받는 별도의 함수를 만들고, 그 함수에 매개변수를 넘겨서 호출

```c++
template<typename T>
class SquareMatrixBase {
protected:
    ...
    void invert(std::size_t matrixsize);
    ...
};

template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
private:
    using SquareMatrixBase<T>::invert;
public:
    ...
    void invert() { this->invert(n); }
};
```

같은 타입의 객체를 원소로 갖는 모든 정방행렬은 오직 한 가지의 SquareMatrixBase 클래스를 공유하게 됨

SquareMatrixBase::invert 함수는 파생 클래스에서 코드 복제를 피할 목적으로만 마련한 장치 → protected 멤버

SquareMatrix와 SquareMatrixBase 사이의 상속 관계는 private → Item 39 참고

---

아직 해결하지 못한 문제가 하나 남았음

SquareMatrixBase::invert 함수는 자신이 상대할 데이터가 어떤 것인지를 어떻게 알 수 있을까?

기본 클래스 쪽에서 역행렬을 만들 수 있도록 '정방행렬의 메모리 위치'를 파생 클래스가 기본 클래스로 넘겨주면 될 것 같은데, 어떻게 할까?

행렬 값을 담는 메모리에 대한 포인터를 SquareMatrixBase가 저장하게 하는 것

```c++
template<typename T>
class SquareMatrixBase {
protected:
    SquareMatrixBase(std::size_t n, T *pMem): size(n), pData(pMem) {}
    void setDataPtr(T *ptr) { pData = ptr; }
    ...
private:
    std::size_t size;  // 행렬의 크기
    T *pData;  // 행렬 값에 대한 포인터
};
```

메모리 할당 방법의 결정 권한이 파생 클래스 쪽으로 넘어가게 됨

```c++
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
public:
    SquareMatrix(): SquareMatrixBase<T>(n, data) {}  // 행렬 크기(n) 및 데이터 포인터를 기본 클래스로 올려보냄
    ...
private:
    T data[n*n];
};
```

이렇게 파생 클래스를 만들면 동적 메모리 할당이 필요 없는 객체가 되지만, 객체 자체의 크기가 커질 수 있음

각 행렬의 데이터를 힙에 둘 수도 있음

```c++
template<typename T, std::size_t n>
class SquareMatrix: private SquareMatrixBase<T> {
public:
    SquareMatrix(): SquareMatrixBase<T>(n, 0), pData(new T[n*n]) { this->setDataPtr(pData.get()); }
    ...
private:
    boost::scoped_array<T> pData;
};
```

어느 메모리에 데이터를 저장하느냐에 따라 설계가 다소 달라짐

SquareMatrix에 속해 있는 멤버 함수 중 상당수가 기본 클래스 버전을 호출하는 단순 인라인 함수가 될 수 있음

똑같은 타입의 데이터를 원소로 갖는 모든 정방행렬들이 행렬 크기에 상관없이 기본 클래스 버전의 사본 하나를 공유함

행렬 크기가 다른 SquareMatrix 객체는 저마다 고유의 타입을 갖고 있음

---

행렬 크기가 미리 녹아든 상태로 별도의 버전이 만들어지는 invert 함수

행렬 크기가 함수 매개변수로 넘겨지거나 객체에 저장된 형태로 다른 파생 클래스들이 공유하는 버전의 invert 함수

전자가 후자보다 더 좋은 코드를 생성할 가능성이 높음

전자의 경우, 행렬 크기가 컴파일 시점에 투입되는 상수이기 때문에 상수 전파(constant propagation) 등의 최적화가 들어가기에 좋음

생성되는 기계 명령어에 대해 이 크기 값이 즉치 피연산자(immediate operand)로 바로 박히는 것도 이런 종류의 최적화 중 하나

여러 행렬 크기에 대해 한 가지 버전의 invert를 두도록 만들면 실행 코드의 크기가 작아지는 이점이 있음

실행 코드가 작아지면 프로그램의 작업 세트 크기가 줄어들면서 명령어 캐시 내의 참조 지역성도 향상됨

두 방법을 전부 적용해보고 그 결과를 관찰

---

객체의 크기

SquareMatrix 객체는 메모리에 생길 때마다 SquareMatrixBase 클래스에 들어 있는 데이터를 가리키는 포인터를 하나씩 떠안고 있음. 파생 클래스 자체에 이미 이 데이터에 접근할 수 있는 수단이 있는데도

SquareMatrix 객체 하나의 크기는 최소한 포인터 하나 크기만큼 낭비된 것

포인터가 필요 없도록 설계를 수정할 수 있지만 잃는 것도 있음

기본 클래스로 하여금 행렬 데이터의 포인터를 protected 멤버로 저장하게끔 만들었다가는 Item 22에서 설명한 캡슐화 효과가 날아가 버림

자원 관리에서도 골치 아픈 일이 생길 수 있음

---

**템플릿을 사용하면 비슷비슷한 클래스와 함수가 여러 벌 만들어짐. 따라서 템플릿 매개변수에 종속되지 않은 템플릿 코드는 비대화의 원인이 됨**

**비타입 템플릿 매개변수로 생기는 코드 비대화의 경우, 템플릿 매개변수를 함수 매개변수 혹은 클래스 데이터 멤버로 대체함으로써 비대화를 종종 없앨 수 있음**

**타입 매개변수로 생기는 코드 비대화의 경우, 동일한 이진 표현구조를 가지고 인스턴스화되는 타입들이 한 가지 함수 구현을 공유하게 만듦으로써 비대화를 감소시킬 수 있음**

