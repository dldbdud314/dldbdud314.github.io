---
title: "여기저기서 긁어 모은 Unity 기본 개념 정리"
categories:
  - Unity
  - TIL
tags:
  - unity
  - C#
author_profile: true
toc: true
---

## [Part 1] Unity
### Canvas와 UI요소
**UI요소란?** Image, Text, Button 등   
- UI요소 생성 시 자동으로 Canvas의 하위 객체로 설정
- UI요소 생성 시 자동으로 EventSystem 객체 생성   
❓ **EventSystem 객체란?** UI의 이벤트를 관리해주는 객체   
❗ UI요소는 Transform 대신 **RectTransform**

#### RectTransform
✔ **앵커, 포지션, 피벗**
1. 앵커
- 부모 오브젝트의 RectTransform에 고정하는 역할
- 해당 오브젝트가 앵커 기준으로 움직이며
- Pos X, Pos Y, Pos Z 값은 앵커 기준으로 설정됨

2. 포지션
- UI 요소가 배치될 위치
- Rect Transform 컴포넌트를 가지고 있는 부모객체를 기준으로 배치

3. 피벗
- 회전시키거나 크기를 바꿀 때 기준이 되는 것

## [Part 2] C#

### 변수
#### 유니티만의 데이터 타입
```
void Start(){
  GameObject monster;
  Transform target;
  Rigidbody myRigidbody;
  Collider myCollider;
}
```
> GameObject : 게임의 오브젝트 저장   
> Transform : 오브젝트의 위치정보 저장   
> Rigidbody : 오브젝트의 물리엔진 정보 저장   
> Collider : 충돌체의 정보 저장

```
GameObject monster1 = GameObject.Find("Cube"); ----- 1
monster1.transform.position = Vector3(0, 0, 0); ---- 2
```
1. 이름이 Cube인 오브젝트의 주소를 참조하는 것
2. 오브젝트의 위치를 선정하는 것

#### var타입과 dynamic타입
- 일반적으로 타입을 명시적으로 선언(int, string, float 등등)
- 명시적 선언으로 처리되지 않길 바라면 쓰는 게 **var 타입** 
- var 타입 변수는 컴파일 시 데이터 타입이 결정된다
- 그리고 한번 초기화한 타입은 변경이 될 수 없다
```
var name = "Hello world"; //name은 string형으로 컴파일 됨
name = 22; //데이터 타입 변경 시 컴파일 에러 발생(데이터 변경 불가)
```

- var 타입의 단점을 보완 = **dynamic 타입**
- 동적으로 데이터 타입을 사용해야 하는 경우
- 위의 에러가 발생하지 않는다   
- **var는 데이터 타입 변경이 안되고, dynamic은 된다**

#### Public & Private field
- public은 인스펙터의 스크립트 위치에 나타나고 수정이 가능함. private은 드러나지 않음!
- 만약 public 필드를 숨기거나 private을 드러내고 싶다면?
```
[HideInIspector]
public int mySpeed = 10;
[SerializeField]
private int yourSpeed = 10;
```

### 함수
#### Start() ⭐⭐⭐
- 게임 시작할 때 한번 호출되는 함수

#### Update() ⭐⭐⭐
- 하나의 **프레임**이 시작될 때 한번씩 호출된다
- 게임 중 지속적으로 발생하는 이동과 같은 변화들을 코딩

#### ※프레임은 뭔데?
- 프레임이란, 하나의 정적인 장면 (프레임이 모여 화면이 동작하는 것처럼 보임)
- 1초에 n개의 프레임을 실행한다 (1초에 몇개의 프레임이 실행됐는지 = FPS: Frame Per Second)

### 컴포넌트 클래스와 일반 클래스
- 게임 오브젝트란? 게임씬에 배치되며 hierarchy뷰에 보이는 객체
- 게임 오브젝트에 부착되는 스크립트 = 컴포넌트 클래스
- 컴포넌트 클래스는 MonoBehaviour 클래스를 상속 받음

### 열거형
아주 유용해보임👀👀
```
enum State {Idle, Walk, Chase, Attack, Dead};
State state = State.Idle;
...
void OnMouseDown(){
  state = State.Dead;
  if(state==State.Dead){
    print("You are Dead");
  }
}
```
-------------------------------------------------------------
### 참고

1. [참고 블로그 1](https://itmining.tistory.com/category/FrontEnd%20%EB%A7%88%EC%9D%B4%EB%8B%9D/Unity)
2. [참고 블로그 2](https://mynameisdabin.tistory.com/31)
3. [참고 블로그 3](https://guslabview.tistory.com/364)
4. [참고 블로그 4](https://m.blog.naver.com/PostView.nhn?blogId=yoohee2018&logNo=220691949344&proxyReferer=https:%2F%2Fwww.google.com%2F)
