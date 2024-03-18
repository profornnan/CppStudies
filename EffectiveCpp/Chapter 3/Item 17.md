# Item 17: new로 생성한 객체를 스마트 포인터에 저장하는 코드는 별도의 한 문장으로 만들자

처리 우선순위를 알려주는 함수 하나와 동적으로 할당된 Widget 객체에 대해 어떤 우선순위에 따라 처리를 적용하는 함수가 하나 있다고 가정

```c++
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
```

자원 관리에는 객체를 사용하는 것이 좋음 → Item 13 참고

동적 할당된 Widget 객체에 대해 스마트 포인터 사용

```c++
processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
```

processWidget 함수 호출

자원 누출 가능성 있음

컴파일러는 processWidget 호출 코드를 만들기 전에 이 함수의 매개변수로 넘겨지는 인자를 평가(evaluate)하는 순서를 밟음

첫 번째 인자는 두 부분으로 나누어져 있기 때문에 processWidget 함수 호출이 이루어지기 전에 컴파일러는 다음의 세 가지 연산을 위한 코드를 만들어야 함

- priority 호출
- "new Widget" 표현식 실행
- tr1::shared_ptr 생성자 호출

각각의 연산이 실행되는 순서는 컴파일러 제작사마다 다름

"new Widget" 표현식은 tr1::shared_ptr 생성자가 실행될 수 있기 전에 호출되어야 함

하지만 priority는 처음 호출될 수도 있고, 두 번째나 세 번째에 호출될 수도 있음

priority가 두 번째에 호출되었고, 예외가 발생했다면 "new Widget"으로 만들어진 포인터가 유실될 수 있음

자원이 누출될 가능성이 있는 이유는, 자원이 생성되는 시점과 그 자원이 자원 관리 객체로 넘어가는 시점 사이에 예외가 끼어들 수 있기 때문

이 문제를 피하는 방법은 Widget을 생성해서 스마트 포인터에 저장하는 코드를 별도의 문장 하나로 만들고, 그 스마트 포인터를 processWidget에 넘기는 것

```c++
std::tr1::shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority());
```

자원 누출 가능성 없음

---

**new로 생성한 객체를 스마트 포인터로 넣는 코드는 별도의 한 문장으로 만들자. 이것이 안 되어 있으면, 예외가 발생될 때 디버깅하기 힘든 자원 누출이 초래될 수 있음**

