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

createInvestment 호출문과 delete가 하나의 루프 안에 들어 있고 continue 혹은 goto 문에 의해 루프에서 빠져 나오는 경우

... 안의 문장에서 예외가 발생한 경우

