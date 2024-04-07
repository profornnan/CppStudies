# Item 20: '값에 의한 전달'보다는 '상수객체 참조자에 의한 전달' 방식을 택하는 편이 대게 낫다

기본적으로 C++는 함수로부터 객체를 전달받거나 함수에 객체를 전달할 때 '값에 의한 전달(pass-by-value)' 방식을 사용함

사본 생성 → 복사 생성자

이 점 때문에 '값에 의한 전달'이 고비용의 연산이 되기도 함

```c++
class Person {
public:
    Person();
    virtual ~Person();
    ...
private:
    std::string name;
    std::string address;
};

class Student: public Person {
public:
    Student();
    ~Student();
    ...
private:
    std::string schoolName;
    std::string schoolAddress;
};
```

```c++
bool validateStudent(Student s);
Student plato;
bool platoIsOK = validateStudent(plato);
```

Student 복사 생성자 호출 한 번, Person 복사 생성자 호출 한 번, String 복사 생선자 호출 네 번

Student 객체를 값으로 전달 시 생성자 여섯 번, 소멸자 여섯 번 호출

---

상수객체에 대한 참조자(reference-to-const)로 전달

```c++
bool validateStudent(const Student& s);
```

새로 만들어지는 객체가 없기 때문에, 생성자와 소멸자가 전혀 호출되지 않음

const가 붙지 않으면 validateStudent 함수로 넘어간 Student 객체가 변할지도 모른다는 걱정을 호출부가 해야 함

---

참조에 의한 전달 방식으로 매개변수를 넘기면 복사손실 문제(slicing problem)가 없어짐

파생 클래스 객체가 기본 클래스 객체로서 전달되는 경우 → 객체가 값으로 전달되면 기본 클래스의 복사 생성자가 호출되고, 파생 클래스 객체로 동작하게 해 주는 특징들이 잘려나감

---

참조자는 보통 포인터를 써서 구현됨

참조자를 전달한다는 것은 결국 포인터를 전달한다는 것

전달하는 객체의 타입이 기본제공 타입일 경우 참조자로 넘기는 것보다 값으로 넘기는 편이 더 효율적일 때가 많음

기본제공 타입, STL 반복자, 함수 객체에 대해서는 '값에 의한 전달'을 선택해도 됨

예전부터 반복자와 함수 객체는 값으로 전달되도록 설계해 왔음

반복자와 함수 객체 구현 시 반드시 ①복사 효율을 높이고 ②복사손실 문제에 노출되지 않도록 만들어야 함

---

기본제공 타입, STL 반복자, 함수 객체 타입 → 값에 의한 전달

이 외의 타입 → 상수객체 참조자에 의한 전달

---

**'값에 의한 전달'보다는 '상수객체 참조자에 의한 전달'을 선호하자. 대체적으로 효율적일 뿐만 아니라 복사손실 문제까지 막아줌**

**기본제공 타입, STL 반복자, 함수 객체는 '값에 의한 전달'이 더 적절함**

