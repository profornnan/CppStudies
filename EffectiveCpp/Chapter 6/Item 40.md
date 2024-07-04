# Item 40: 다중 상속은 심사숙고해서 사용하자

다중 상속(multiple inheritance: MI)

다중 상속을 하면 둘 이상의 기본 클래스로부터 똑같은 이름(함수, typedef 등)을 물려받을 가능성이 생김

다중 상속 때문에도 모호성이 생긴다는 것

```c++
class BorrowableItem {
public:
    void checkOut();
    ...
};

class ElectronicGadget {
private:
    bool checkOut() const;
    ...
};

class MP3Player: public BorrowableItem, public ElectronicGadget { ... };

MP3Player mp;
mp.checkOut();  // 모호성 발생
```

두 checkOut 함수들 중에서 파생 클래스가 접근할 수 있는 함수가 딱 결정되는 것이 분명한데도(BorrowableItem에서는 public 멤버, ElectronicGadget에서는 private 멤버) 모호성이 생김

중복된 함수 호출 중 하나를 골라내는 C++의 규칙을 따른 결과

최적 일치 함수를 찾은 후에 함수의 접근가능성 점검

두 checkOut 함수는 C++ 규칙에 의한 일치도가 서로 같기 때문에, 최적 일치 함수가 결정되지 않음

그렇기 때문에 ElectronicGadget::checkOut 함수의 접근가능성이 점검되는 순서조차 오지 않음

---

모호성을 해소하려면, 호출할 기본 클래스의 함수를 지정해 주어야 함

```c++
mp.BorrowableItem::checkOut();
```

---

다중 상속의 의미는 둘 이상의 클래스로부터 상속을 받는 것

MI는 상위 단계의 기본 클래스를 여러 개 갖는 클래스 계통에서 심심치 않게 눈에 띔

죽음의 MI 마름모꼴(deadly MI diamond)이라고 알려진 좋지 않은 모양이 나올 수 있음

```c++
class File { ... };
class InputFile: public File { ... };
class OutputFile: public File { ... };
class IOFile: public InputFile, public OutputFile { ... };
```

기본 클래스와 파생 클래스 사이의 경로가 두 개 이상이 되는 상속 계통 → 기본 클래스의 데이터 멤버가 경로 개수만큼 중복 생성됨

기본적으로는 데이터 멤버를 중복생성

만약 데이터 멤버의 중복생성을 원한 것이 아니었다면, 해당 데이터 멤버를 가진 클래스를 가상 기본 클래스(virtual base class)로 만드는 것으로 해결 가능

가상 기본 클래스로 삼을 클래스에 직접 연결된 파생 클래스에서 가상 상속(virtual inheritance)을 사용하게 만드는 것

```c++
class File { ... };
class InputFile: virtual public File { ... };
class OutputFile: virtual public File { ... };
class IOFile: public InputFile, public OutputFile { ... };
```

표준 C++ 라이브러리가 이런 모양의 MI 상속 계통을 하나 갖고 있음

클래스 템플릿

basic_ios, basic_istream, basic_ostream, basic_iostream

---

정확한 동작의 관점에서 보면, public 상속은 반드시 항상 가상 상속이어야 함

정확성 외에 다른 측면도 같이 생각

가상 상속은 비쌈(크기 ↑, 속도 ↓)

가상 기본 클래스의 초기화에 관련된 규칙은 비가상 기본 클래스의 초기화 규칙보다 훨씬 복잡한데다가 직관성도 더 떨어짐

대부분의 경우, 가상 상속이 되어 있는 클래스 계통에서는 파생 클래스들로 인해 가상 기본 클래스 부분을 초기화할 일이 생기게 됨

초기화 규칙

1. 초기화가 필요한 가상 기본 클래스로부터 클래스가 파생된 경우, 이 파생 클래스는 가상 기본 클래스와의 거리에 상관없이 가상 기본 클래스의 존재를 염두에 두고 있어야 함
2. 기존의 클래스 계통에 파생 클래스를 새로 추가할 때도 그 파생 클래스는 가상 기본 클래스(역시 거리에 상관없이)의 초기화를 떠맡아야 함

---

구태여 쓸 필요가 없으면 가상 기본 클래스를 사용하지 말자. 비가상 상속을 기본으로 삼자

가상 기본 클래스를 정말 쓰지 않으면 안 될 상황이라면, 가상 기본 클래스에는 데이터를 넣지 않는 쪽으로 최대한 신경쓰자

---

C++ 인터페이스 클래스를 써서 사람 모형화 → Item 31 참고

```c++
class IPerson {
public:
    virtual ~IPerson();
    virtual std::string name() const = 0;
    virtual std::string birthDate() const = 0;
};
```

IPerson을 쓰려면 IPerson 포인터 및 참조자를 통해 프로그래밍을 해야할 것임

조작이 가능한 IPerson 객체를 생성하기 위해, IPerson의 사용자는 팩토리 함수를 사용해서 IPerson의 구체 파생 클래스를 인스턴스로 만듦

```c++
// 유일한 데이터베이스 ID로부터 IPerson 객체를 만들어내는 팩토리 함수 → Item 18 참고
std::tr1::shared_ptr<IPerson> makePerson(DatabaseID personIdentifier);

// 사용자로부터 데이터베이스 ID를 얻어내는 함수
DatabaseID askUserForDatabaseID();

DatabaseID id(askUserForDatabaseID());
// IPerson 인터페이스를 지원하는 객체를 하나 만들고 pp로 가리키게 함
std::tr1::shared_ptr<IPerson> pp(makePerson(id));
// 이후에는 *pp의 조작을 위해 IPerson의 멤버 함수 사용
...
```

makePerson 함수가 인스턴스로 만들 수 있는 구체 클래스가 IPerson으로부터 파생되어 있어야 할 것임

이 클래스의 이름이 CPerson이라고 가정

CPerson은 IPerson으로부터 물려받은 순수 가상 함수에 대한 구현을 제공해야 함

```c++
class PersonInfo {
public:
    explicit PersonInfo(DatabaseID pid);
    virtual ~PersonInfo();
    virtual const char * theName() const;
    virtual const char * theBirthDate() const;
    ...
private:
    virtual const char * valueDelimOpen() const;
    virtual const char * valueDelimClose() const;
    ...
};
```

PersonInfo 클래스

데이터베이스 필드를 다양한 서식으로 출력할 수 있는 기능

기본적으로, 출력용 필드 값의 시작과 끝에 붙는 구분자가 대괄호([])로 미리 정해져 있음

사용자가 원하는 시작 구분자와 끝 구분자를 파생 클래스에서 지정할 수 있도록 valueDelimOpen 함수와 valueDelimClose 함수를 가상 함수로 마련해 둠

PersonInfo 클래스의 다른 멤버 함수들은 이 가상 함수를 통해 자신들이 사용하는 필드 값에 적절한 구분자를 붙이도록 구현되는 것

theName은 valueDelimOpen을 호출해서 시작 구분자를 만들고, name 값 자체를 만든 다음, valueDelimClose를 호출하도록 구현됨

valueDelimOpen, valueDelimClose는 가상 함수이기 때문에, theName이 반환하는 결과는 PersonInfo에만 좌우되는 것이 아니라 PersonInfo로부터 파생된 클래스에도 좌우됨

---

CPerson을 구현하는 사람의 입장에서 볼 때 이 점은 아주 반가운 소식

IPerson 문서 → name과 birthDate 함수가 반환하는 값에는 구분자가 붙으면 안 됨

---

CPerson과 PersonInfo는 is-implemented-in-terms-of 관계

이 관계를 표현하는 방법은 객체 합성과 private 상속

대부분의 경우 객체 합성 선호

가상 함수를 꼭 재정의해야 한다면 private 상속을 써야 함

CPerson이 PersonInfo로부터 private 상속을 받도록 만드는 것

CPerson 클래스는 IPerson 인터페이스도 함께 구현해야 함 → public 상속

인터페이스의 public 상속과 구현의 private 상속 조합

```c++
class IPerson { ... };
class DatabaseID { ... };
class PersonInfo { ... };

class CPerson: public IPerson, private PersonInfo {
public:
    explicit CPerson(DatabaseID pid): PersonInfo(pid) {}
    virtual std::string name() const { return PersonInfo::theName(); }
    virtual std::string birthDate() const { return PersonInfo::theBirthDate(); }
private:
    const char * valueDelimOpen() const { return ""; }
    const char * valueDelimClose() const { return ""; }
};
```

---

다중 상속은 객체 지향 기법으로 소프트웨어를 개발하는 데 쓰이는 도구 중 하나

---

**다중 상속은 단일 상속보다 확실히 복잡함. 새로운 모호성 문제를 일으킬 뿐만 아니라 가상 상속이 필요해질 수도 있음**

**가상 상속을 쓰면 크기 비용, 속도 비용이 늘어나며, 초기화 및 대입 연산의 복잡도가 커짐. 따라서 가상 기본 클래스에는 데이터를 두지 않는 것이 현실적으로 가장 실용적임**

**다중 상속을 적법하게 쓸 수 있는 경우가 있음. 인터페이스 클래스로부터 public 상속을 시킴과 동시에 구현을 돕는 클래스로부터 private 상속을 시키는 것**

