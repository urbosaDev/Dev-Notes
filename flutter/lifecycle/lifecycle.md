
# 📘 Flutter 위젯 → Element → RenderObject 흐름 정리

---

## ✅ Widget

- 모든 위젯은 `abstract class Widget`을 상속받는다.
- Widget class 내부에는 `Element createElement()` 가 있다.
- Widget은 그 자체로 화면을 표현하지 않는다.
- 대신 `createElement()` 를 통해 자신을 실제 UI에 연결할 수 있는 Element를 만들어낸다.

---

## ✅ 그럼 Element가 뭐지?

- Element는 `BuildContext`를 implements한다.
- BuildContext는 Widget이 자신의 부모, 형제, 자식들과 소통할 수 있도록  
  Flutter가 내부적으로 만들어주는 **Element에 대한 인터페이스**이다 (추상화, 다형성).

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const Placeholder();
  }
}
```

- Widget은 Widget class를 배출하는 build 메서드를 사용한다.
- 파라미터 `(BuildContext context)`는 위젯의 위치값과 내부 내용을 갖고 있는 Element이다.
- build(BuildContext context)는 Widget이 자신의 UI 하위 구조를 정의하는 함수이다.

---

## ✅ build 메서드는 누가 실행할까?

```dart
abstract class StatelessWidget extends Widget {
  Widget build(BuildContext context); // 너가 override 하는 메서드

  @override
  StatelessElement createElement() => StatelessElement(this);
}

class StatelessElement extends ComponentElement {
  @override
  Widget build() => widget.build(this); // ← 요거!
}
```

- build 메서드를 호출하는 주체는 Element이다.
- 즉, Widget Tree를 만드는 건 build() 메서드이다. 그리고 주체는 Element이다.

---

## ✅ 다시 한 번 정리해보자

- Widget 자체는 껍데기일 뿐임.
- Widget Tree는 build 메서드에 context 파라미터를 받아서, 위치값을 가진 채로 생성이 됨 (build 메서드의 return 값은 Widget).
- Element Tree는 위젯을 생성할 때 생성됨 (class MyApp extends StatelessWidget).
- StatelessWidget 내부에 createElement()가 있고, 이를 통해 위젯의 정보를 가진 Element가 탄생함.
- 위젯의 정보값을 가졌으니 당연히 Tree 구조.
- build 메서드를 사용할 때마다 사용한 주체는 Element임. 그리고 build 메서드를 사용할 때마다 Widget Tree가 재구성되는 것임.

---

## ✅ 그림으로 한번 표현해보자


![alt text](./assets/IMG_ED3CA2021E6E-1.jpg)

---

## ✅ 그럼 이제 Element와 RenderObject의 관계를 보자

- Text, Column, Container 같은 위젯들은 RenderObjectWidget을 상속한다.

```
                   ┌──────────────┐
                   │  Widget (추상) │  ← 모든 위젯의 부모
                   └──────┬───────┘
                          ↓
   ┌─────────────────────────────┬─────────────────────────────┐
   │                             │                             │
   ↓                             ↓                             ↓
StatelessWidget           StatefulWidget           RenderObjectWidget ← 요거 주목!
   ↓                             ↓                             ↓
(createElement)           (createElement)           (createRenderObject)
   ↓                             ↓                             ↓
StatelessElement          StatefulElement          RenderObjectElement
                             ↓
                          State<T>
                             ↓ build()
                           Widget Tree
```

---

## ✅ 하지만 Container는?

- Container 자체는 StatelessWidget을 상속받은 껍데기이다.
- 어떻게 RenderObjectWidget을 상속받을까?

### 간접적으로 상속을 받는다

예를 들어 Container 내부에 `padding`, `decoration` 등은  
`EdgeInsets`, `BoxDecoration` 등의 데이터 모델을 가진다.  
그리고 이들은 그 값들을 받아서 처리하는 RenderObject들이 내부에 생성된다.  
즉, RenderObjectWidget 들을 조합하여 반환한다.  
고로 간접적으로 상속을 받는다.

---

## ✅ 위젯 생성 흐름 정리

- 우리가 위젯을 생성하면 (Container를 생성할 때 StatelessWidget을 상속),
- 그 내부는 width: ... 값들을 넣는다.
- 그 값들은 context를 통해 Element가 갖게 된다 (createElement).
- → context는 Element 자체고,
- 위젯에 넣은 생성자 값들은 Element의 widget 필드에 저장된다.
- 그리고 그 과정에서 widget.createRenderObject(context) 호출할 때,  
  그 값을 기반으로 RenderObject가 만들어진다.

---

## ✅ Element의 역할 요약

- Element는 Widget에 의해 생성되고 (위치와, 생김새의 데이터를 받아서),  
  그 값을 RenderObject에 주입하고 화면에 적용한다.

---

## ✅ 시각 요약

![alt text](./assets/IMG_ED3CA2021E6E-1.jpg)

---

## ✅ 요약 표

| 구성 요소 | 설명 |
|-----------|------|
| Widget | 선언 및 구조 설계 (껍데기) |
| Element | 상태 및 생명주기 관리자, context 역할 |
| RenderObject | 실제 UI를 그리고 레이아웃을 처리하는 객체 |

---

> **🧠 한 문장 요약**  
> Widget은 선언, Element는 연결자, RenderObject는 실제 그림을 그리는 주체이다.  
> 이 세 가지가 함께 Flutter의 위젯 렌더링 구조를 이룬다.
