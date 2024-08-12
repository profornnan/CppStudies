# Item 43: 템플릿으로 만들어진 기본 클래스 안의 이름에 접근하는 방법을 알아 두자

다른 몇 개의 회사에 메시지를 전송할 수 있는 응용프로그램

전송용 메시지 형태 → 암호화, 비암호화

템플릿 기반 방법 사용

```c++
class CompanyA {
public:
    ...
    void sendCleartext(const std::string& msg);
    void sendEncrypted(const std::string& msg);
    ...
};

class CompanyB {
public:
    ...
    void sendCleartext(const std::string& msg);
    void sendEncrypted(const std::string& msg);
    ...
};

...  // 다른 회사들을 나타내는 각각의 클래스

class MsgInfo { ... };  // 메시지 생성에 사용되는 정보를 담기 위한 클래스

template<typename Company>
class MsgSender {
public:
    ...  // 생성자, 소멸자 등
    void sendClear(const MsgInfo& info) {
        std::string msg;
        info로부터 msg를 만듦;
        
        Company c;
        c.sendCleartext(msg);
    }
    
    void sendSecret(const MsgInfo& info) { ... }
};
```

메시지를 보낼 때마다 관련 정보를 로그로 남기기

파생 클래스 사용

```c++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...  // 생성자, 소멸자 등
    void sendClearMsg(const MsgInfo& info) {
        메시지 전송 전 정보를 로그에 기록;
        sendClear(info);  // 기본 클래스의 함수를 호출하는데, 이 코드는 컴파일되지 않음
        메시지 전송 후 정보를 로그에 기록;
    }
    ...
};
```

이 코드는 컴파일되지 않음

sendClear 함수가 존재하지 않는다는 것이 컴파일이 안 되는 이유

컴파일러가 LoggingMsgSender 클래스 템플릿의 정의와 마주칠 때, 컴파일러는 이 클래스가 어디서 파생된 것인지 모름

Company는 템플릿 매개변수 → LoggingMsgSender가 인스턴스로 만들어질 때까지 무엇이 될지 알 수 없음

---

암호화된 통신만을 사용해야 하는 CompanyZ 클래스

```c++
class CompanyZ {
public:
    ...
    void sendEncrypted(const std::string& msg);
    ...
};
```

CompanyZ를 위한 MsgSender의 특수화 버전

```c++
template<>
class MsgSender<CompanyZ> {  // MsgSender 템플릿의 완전 특수화 버전
public:
    ...
    void sendSecret(const MsgInfo& info) { ... }
};
```

template\<\> → 이건 템플릿도 아니고 클래스도 아니라는 것

위의 코드는 MsgSender 템플릿을 템플릿 매개변수가 CompanyZ일 때 쓸 수 있도록 특수화한 버전

**완전 템플릿 특수화(total template specialization)**

MsgSender 템플릿이 CompanyZ 타입에 대해 특수화됨

템플릿의 매개변수들이 하나도 빠짐없이 구체적인 타입으로 정해진 상태

---

MsgSender\<CompanyZ\> 클래스에는 sendClear 함수가 없기 때문에 LoggingMsgSender의 기본 클래스가 MsgSender\<CompanyZ\>이면 말이 되지 않음

기본 클래스 템플릿은 언제라도 특수화될 수 있음

특수화 버전에서 제공하는 인터페이스가 원래의 일반형 템플릿과 꼭 같으리란 법은 없음

이렇기 때문에, C++ 컴파일러는 템플릿으로 만들어진 기본 클래스를 뒤져서 상속된 이름을 찾는 것을 거부함

---

세 가지 해결 방법

첫째, 기본 클래스 함수에 대한 호출문 앞에 this-\>를 붙임

```c++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...
    void sendClearMsg(const MsgInfo& info) {
        메시지 전송 전 정보를 로그에 기록;
        this->sendClear(info);  // sendClear가 상속되는 것으로 가정함
        메시지 전송 후 정보를 로그에 기록;
    }
    ...
};
```

둘째, using 선언 사용

```c++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    using MsgSender<Company>::sendClear;  // 컴파일러에게 sendClear 함수가 기본 클래스에 있다고 가정하라고 알려줌
    ...
    void sendClearMsg(const MsgInfo& info) {
        ...
        sendClear(info);  // sendClear가 상속되는 것으로 가정함
        ...
    }
    ...
};
```

셋째, 호출할 함수가 기본 클래스의 함수라는 점을 명시적으로 지정

```c++
template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
    ...
    void sendClearMsg(const MsgInfo& info) {
        ...
        MsgSender<Company>::sendClear(info);  // sendClear가 상속되는 것으로 가정함
        ...
    }
    ...
};
```

마지막 방법은 추천하지 않음

호출되는 함수가 가상 함수인 경우에는, 이런 식으로 명시적 한정을 해 버리면 가상 함수 바인딩이 무시되기 때문

---

기본 클래스 템플릿이 이후에 어떻게 특수화되더라도 원래의 일반형 템플릿에서 제공하는 인터페이스를 그대로 제공할 것이라고 컴파일러에게 약속하는 것

---

기본 클래스의 멤버에 대한 참조가 무효한지를 컴파일러가 진단하는 과정이 파생 클래스 템플릿의 정의가 구문분석될 때 들어감

이른 진단

---

**파생 클래스 템플릿에서 기본 클래스 템플릿의 이름을 참조할 때는 this-\>를 접두사로 붙이거나 기본 클래스 한정문을 명시적으로 써주는 것으로 해결**

