# Item 31: 파일 사이의 컴파일 의존성을 최대로 줄이자

C++는 인터페이스와 구현을 깔끔하게 분리하는 일에 별로 일가견이 없음

C++의 클래스 정의는 클래스 인터페이스만 지정하는 것이 아니라 구현 세부사항까지 상당히 많이 지정하고 있음

```c++
class Person {
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::string theName;  // 구현 세부사항
    Date theBirthDate;  // 구현 세부사항
    Address theAddress;  // 구현 세부사항
};
```

위의 코드만 가지고 Person 클래스 컴파일 불가능

Person의 구현 세부사항에 속하는 것들(string, Date, Address)이 어떻게 정의됐는지를 모르면 컴파일 자체가 불가능함

#include 지시자를 사용해 이들이 정의된 정보를 가져옴

```c++
#include <string>
#include "date.h"
#include "address.h"
```

#include문은 Person을 정의한 파일과 위의 헤더 파일들 사이에 컴파일 의존성(compilation dependency)을 엮음

위의 헤더 파일, 이들과 엮여 있는 헤더 파일들이 바뀌기만 해도 Person 클래스를 정의한 파일은 다시 컴파일됨

Person을 사용하는 다른 파일들까지 전부 다시 컴파일됨

꼬리에 꼬리를 무는 컴파일 의존성이 있으면 프로젝트가 고통스러워짐

---

Person 클래스 정의 시 구현 세부사항을 따로 떼어서 지정

```c++
namespace std {
    class string;  // 전방 선언(틀림)
}

class Date;  // 전방 선언
class Address;  // 전방 선언

class Person {
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
};
```

필요한 요소들을 모두 전방 선언

두 가지 문제가 있음

string은 클래스가 아니라 typedef로 정의한 타입동의어임(basic_string\<char>를 typedef한 것) → 제대로 전방 선언을 하려면 템플릿을 추가로 끌고 들어와야 하기 때문에 더 복잡함

컴파일러가 컴파일 도중에 객체들의 크기를 전부 알아야 함

---

포인터 뒤에 실제 객체 구현부 숨기기

클래스를 두 클래스로 쪼개기

한쪽은 인터페이스만 제공하고, 또 한쪽은 그 인터페이스의 구현을 맡도록 만들기

구현을 맡은 클래스 → PersonImpl

```c++
#include <string>  // 표준 라이브러리 구성요소는 전방 선언하면 안 됨
#include <memory>  // tr1::shared_ptr을 위한 헤더

class PersonImpl;  // Person의 구현 클래스에 대한 전방 선언

class Date;  // Person 클래스 안에서 사용되는 것들에 대한 전방 선언
class Address;

class Person {
public:
    Person(const std::string& name, const Date& birthday, const Address& addr);
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    ...
private:
    std::tr1::shared_ptr<PersonImpl> pImpl;  // 구현 클래스 객체에 대한 포인터
};
```

주 클래스(Person)에 들어 있는 데이터 멤버는 구현 클래스(PersonImpl)에 대한 포인터뿐임

이런 설계는 거의 패턴으로 굳어져 있음 → pimpl 관용구(pointer to implementation)

---

Person 클래스에 대한 구현 클래스 부분을 고쳐도 Person의 사용자 쪽에서는 컴파일을 다시 할 필요가 없음

인터페이스와 구현이 분리됨

---

인터페이스와 구현을 둘로 나누는 열쇠는 '정의부에 대한 의존성'을 '선언부에 대한 의존성'으로 바꾸어 놓는 데 있음

컴파일 의존성을 최소화하는 핵심 원리

헤더 파일을 만들 때는 실용적인 의미를 갖는 한 자체조달(self-sufficient) 형태로 만들고, 정 안 되면 다른 파일에 대해 의존성을 갖도록 하되 정의부가 아닌 선언부에 대해 의존성을 갖도록 만드는 것

### 객체 참조자 및 포인터로 충분한 경우에는 객체를 직접 쓰지 않기

어떤 타입에 대한 참조자 및 포인터를 정의할 때는 그 타입의 선언부만 필요함

반면, 어떤 타입의 객체를 정의할 때는 그 타입의 정의가 준비되어 있어야 함

### 할 수 있으면 클래스 정의 대신 클래스 선언에 최대한 의존하도록 만들기

어떤 클래스를 사용하는 함수를 선언할 때는 그 클래스의 정의를 가져오지 않아도 됨

심지어 그 클래스 객체를 값으로 전달하거나 반환하더라도 클래스 정의가 필요 없음

클래스 정의를 제공하는 일을 함수 선언이 되어 있는 헤더 파일 쪽에 주지 않고 실제 함수 호출이 일어나는 사용자의 소스 파일 쪽에 전가하는 방법 사용

실제로 쓰지도 않을 타입 정의에 대해 사용자가 의존성을 끌어오는 것을 막을 수 있음

### 선언부와 정의부에 대해 별도의 헤더 파일을 제공하기

클래스를 둘로 쪼개자는 지침을 제대로 쓸 수 있도록 하려면 헤더 파일이 짝으로 있어야 함

선언부를 위한 헤더 파일, 정의부를 위한 헤더 파일

한쪽에서 어떤 선언이 바뀌면 다른 쪽도 똑같이 바꿔야 함

라이브러리 사용자 쪽에서는 전방 선언 대신에 선언부 헤더 파일을 항상 #include해야 하고, 라이브러리 제작자 쪽에서는 헤더 파일 두 개를 짝지어 제공해야 함

---

C++에서는 템플릿 선언과 템플릿 정의를 분리할 수 있도록 하는 기능을 export 키워드로 제공하고 있음

현재 이 키워드를 제대로 지원하는 컴파일러가 별로 없고, export를 현장에서 쓰는 예가 너무 드물다는 것이 문제

---

pimpl 관용구를 사용하는 Person 같은 클래스를 핸들 클래스(handle class)라고 함

핸들 클래스에서 어떤 함수를 호출하게 되어 있다면, 핸들 클래스에 대응되는 구현 클래스 쪽으로 그 함수 호출을 전달해서 구현 클래스가 실제 작업을 수행하게 만들자

