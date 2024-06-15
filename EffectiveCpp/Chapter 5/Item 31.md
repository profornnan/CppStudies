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

---

Person을 특수 형태의 추상 기본 클래스, 이른 바 인터페이스 클래스(Interface class)로 만드는 방법도 생각해 볼 수 있음

어떤 기능을 나타내는 인터페이스를 추상 기본 클래스를 통해 마련해 놓고, 이 클래스로부터 파생 클래스를 만들 수 있게 하는 것 → Item 34 참고

파생이 목적이기 때문에 데이터 멤버도 없고, 생성자도 없으며, 하나의 가상 소멸자(Item 7 참고)와 인터페이스를 구성하는 순수 가상 함수만 들어 있음

비가상 함수의 구현은 주어진 클래스 계통 내의 모든 클래스에 대해 똑같아야 함 → Item 36 참고

따라서 비가상 함수는 인터페이스 클래스의 일부로서 구현해 두는 편이 이치에 맞음

```c++
class Person {
public:
    virtual ~Person();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
    virtual std::string address() const = 0;
    ...
};
```

순수 가상 함수를 포함한 클래스를 인스턴스로 만들기는 불가능함 → Person에 대한 포인터 혹은 참조자로 프로그래밍

인터페이스 클래스의 인터페이스가 수정되지 않는 한 사용자는 다시 컴파일할 필요가 없음

---

인터페이스 클래스를 사용하기 위해서는 객체 생성 수단이 최소한 하나는 있어야 함

대개 이 문제는 파생 클래스의 생성자 역할을 대신하는 어떤 함수를 만들어 놓고 이것을 호출함으로써 해결함

팩토리 함수(Item 13 참고) 혹은 가상 생성자(virtual constructor)

주어진 인터페이스 클래스의 인터페이스를 지원하는 객체를 동적으로 할당한 후, 그 객체의 포인터를 반환

이런 함수는 인터페이스 클래스 내부에 정적 멤버로 선언되는 경우가 많음

```c++
class Person {
public:
    ...
    static std::tr1::shared_ptr<Person> create(const std::string& name, const Date& birthday, const Address& addr);
    ...
};
```

사용자 쪽에서는 다음과 같이 사용

```c++
std::string name;
Date dateOfBirth;
Address address;
...

// Person 인터페이스를 지원하는 객체 한 개 생성
std::tr1::shared_ptr<Person> pp(Person::create(name, dateOfBirth, address));
...
std::cout << pp->name() << " was born on " << pp->birthDate() << " and now lives at " << pp->address();
```

해당 인터페이스 클래스의 인터페이스를 지원하는 구체 클래스(concrete class)가 어디엔가 정의되어야 하고 구체 클래스의 생성자가 호출되어야 함 → 가상 생성자의 구현부를 갖고 있는 파일 안에서 이뤄짐

---

Person 클래스로부터 파생된 RealPerson이라는 구체 클래스가 있다면, 이 클래스는 자신이 상속받은 가상 함수(순수 가상 함수)에 대한 구현부를 제공하는 식으로 만들어졌을 것임

```c++
class RealPerson: public Person {
public:
    RealPerson(const std::string& name, const Date& birthday, const Address& addr): theName(name), theBirthDate(birthday), theAddress(addr) {}
    
    virtual ~RealPerson() {}
    
    std::string name() const;
    std::string birthDate() const;
    std::string address() const;
    
private:
    std::string theName;
    Date theBirthDate;
    Address theAddress;
};
```

Person::create 함수

```c++
std::tr1::shared_ptr<Person> Person::create(const std::string& name, const Date& birthday, const Address& addr) {
    return std::tr1::shared_ptr<Person>(new RealPerson(name, birthday, addr));
}
```

---

인터페이스 클래스를 구현하는 용도로 가장 많이 쓰는 메커니즘이 두 가지 있는데, RealPerson 예제는 그 중 하나

인터페이스 클래스로부터 인터페이스 명세를 물려받게 만든 후에, 그 인터페이스에 들어 있는 함수(가상 함수)를 구현하는 것

두 번째 방법은 다중 상속을 하는 것 → Item 40 참고

---

핸들 클래스와 인터페이스 클래스는 구현부로부터 인터페이스를 떼어 놓음으로써 파일들 사이의 컴파일 의존성을 완화시키는 효과를 가져다줌

실행 시간 비용이 들어감

객체 한 개당 필요한 저장 공간이 추가로 늘어남

---

핸들 클래스의 멤버 함수를 호출하면 구현부 객체의 데이터까지 가기 위해 포인터를 타야 함 → 요구되는 간접화 연산이 한 단계 더 증가

객체 하나씩을 저장하는데 필요한 메모리 크기에 구현부 포인터의 크기가 더해짐

구현부 포인터가 동적 할당된 구현부 객체를 가리키도록 어디선가 그 구현부 포인터의 초기화가 일어나야 함(핸들 클래스의 생성자 안에서)

---

인터페이스 클래스의 경우에는 호출되는 함수가 전부 가상 함수라는 것이 약점임

함수 호출이 일어날 때마다 가상 테이블 점프에 따르는 비용이 소모됨 → Item 7 참고

인터페이스 클래스에서 파생된 객체는 전부 가상 테이블 포인터를 지니고 있어야 함 → Item 7 참고

만약 가상 함수를 공급하는 쪽이 인터페이스 클래스밖에 없을 때는, 이 가상 테이블 포인터도 객체 하나를 저장하는 데 필요한 메모리 크기를 늘리는 요인이 됨

---

핸들 클래스와 인터페이스 클래스는 인라인 함수의 도움을 제대로 끌어내기 힘듦

---

개발 도중에는 핸들 클래스 혹은 인터페이스 클래스를 사용

구현부가 바뀌었을 때 사용자에게 미칠 파급 효과를 최소로 만드는 것이 좋음

제품을 출시해야 될 때 다시 고민

---

**컴파일 의존성을 최소화하는 작업의 배경이 되는 가장 기본적인 아이디어는 정의 대신에 선언에 의존하게 만들자는 것. 이 아이디어에 기반한 두 가지 접근 방법은 핸들 클래스와 인터페이스 클래스**

**라이브러리 헤더는 그 자체로 모든 것을 갖추어야 하며 선언부만 갖고 있는 형태여야 함. 이 규칙은 템플릿이 쓰이거나 쓰이지 않거나 동일하게 적용**

