---
title: "unity에서 firebase 실시간 데이터 읽기"
categories:
  - Unity
tags:
  - firebase
  - unity
author_profile: true
toc: true
---

## #️⃣ 준비

데이터를 읽어들이기 위해선 아래 과정이 선행 되어야 합니다.

✔ Unity 프로젝트를 등록하고 Firebase를 사용하도록 구성하기   
✔ Firebase Unity SDK를 Unity 프로젝트에 추가하기

이를 마쳤다면 본격적으로 데이터를 읽어들이는 코드를 다뤄 보겠습니다.

database가 아래와 같이 구성되어 있다고 가정합시다.
```
reference_one
ㄴ0
  ㄴname: "Joy"
  ㄴage: 21
ㄴ1
  ㄴname: "Lara"
  ㄴage: 26
ㄴ2
  ㄴname: "Jake"
  ㄴage: 34
```

## 0️⃣ Database reference 가져오기
우선 데이터를 가져오기 위해서는 레퍼런스를 지정해줘야 합니다.
```
using Firebase;
using Firebase.Database;
using Firebase.Unity.Editor;

FirebaseApp.DefaultInstance.SetEditorDatabaseUrl(해당 firebase 주소);
DatabaseReference reference = FirebaseDatabase.DefaultInstance.GetReference("reference_one");
```

## 1️⃣ 데이터 읽는 방식
**일회성으로 읽어 들이는지, 혹은 지속적으로 읽어 들이는지**에 따라 크게 두가지 방식으로 나눌 수 있습니다.   
1. 데이터 한번 읽기
해당 방식은 지정된 경로에서 해당 위치의 모든 데이터를 포함하는 정적 스냅샷(DataSnapshot)을 찍습니다. 데이터를 읽어들이면 DataSnapshot의 형태로 이를 받을 수 있습니다.     

2. 이벤트 수신 대기
이벤트 리스너를 추가하는 방식으로, 데이터의 변화가 있을 때마다 이를 읽어들일 수 있습니다.   
아래와 같은 이벤트들에 리스너를 추가할 수 있습니다.  
![image](https://user-images.githubusercontent.com/57944099/99770279-eedd8000-2b4a-11eb-8ca7-87a9df1c01ef.png)
- 이러한 이벤트는 리스너가 연결될 때 **한 번 호출된 후** 하위 데이터를 포함한 **데이터가 변경될 때마다 다시 호출**됩니다.   

두 방식 모두 DataSnapshot을 활용하여 데이터를 처리합니다.    
👇 아래는 DataSnapshot에 해당 하는 다양한 메소트와 프로퍼티에 대한 공식 문서인데 참고하시면 되겠습니다.   
[DataSnapshot](https://firebase.google.com/docs/reference/android/com/google/firebase/database/DataSnapshot)

## 2️⃣ 데이터 한번 읽기
- GetValueAsync()를 활용하여 한번 읽을 수 있습니다. 이는 **비동기 메소드**라는 점을 유의해서 사용해주세요!! 이는 아래 2-1로 더 구체적으로 다뤄 보겠습니다.
```
reference.GetValueAsync().ContinueWith(task => {
     if (task.IsFaulted) {
       Debug.LogError("failed reading...");
       return;
     }
     else if (task.IsCompleted) {
       DataSnapshot snapshot = task.Result;
       // 스냅샷을 이용하여 처리하기
     }
});
```

아래는 스냅샷을 이용한 처리부의 예로, 다수의 데이터를 저장하는 예시입니다.
```
foreach(DataSnapshot data in snapshot.Children){
    IDictionary people = (IDictionary)data.Value;
    Debug.Log("이름: " + people["name"] + ", 나이: " + people["age"]);
}
```
- firebase의 데이터베이스는 json 형태로, 딕셔너리와 같이 key-value쌍으로 이루어져 있습니다.
- Key 프로퍼티와 Value 프로퍼티를 활용하여 접근할 수 있습니다.

### 2️⃣-1️⃣ 비동기 처리
- 비동기 메소드는 반드시 순차적으로 실행되지 않습니다. 순차적으로 실행되는 것을 보장하기 위해서는 **async-await**을 활용해 봅시다.   
✔ **How?**
   * 오래 걸릴 가능성이 있는 비동기 메소드 앞에는 await 키워드를 붙이며, 해당 함수가 포함되어 있는 함수 앞에는 async 키워드를 붙여서 동기적으로 처리될 수 있도록 합니다.
   * async 함수는 반드시 반환형이 void 또는 Task여야 합니다.
   * 만약 반환형이 있는 함수라면, 그 반환형이 int인 경우 Task<int>와 같은 식으로 반환할 수 있습니다.
- 위 함수를 동기적으로 처리하는 방식으로 바꿔보겠습니다. 만약 위 함수가 Start()에서 호출된다면,   

```
async void Start(){
  await reference.GetValueAsync().ContinueWith(task => {
     if (task.IsFaulted) {
       Debug.LogError("failed reading...");
       return;
     }
     else if (task.IsCompleted) {
       DataSnapshot snapshot = task.Result;
       // 스냅샷을 이용하여 처리하기
     }
  });
}
```

이런식으로 처리할 수 있습니다.

## 3️⃣ 이벤트 수신 대기
- ChildAdded 이벤트에 대한 리스너

```
reference.ChildAdded += (object sender, ChildChangedEventArgs args) =>
 {
      if (args.DatabaseError != null)
      {
          Debug.LogError(args.DatabaseError.Message);
          return;
      }

      IDictionary people = (IDictionary)args.Snapshot.Value;
      Debug.Log("이름: " + people["name"] + ", 나이: " + people["age"]);
};
```

- 만약 reference보다 더 깊은 뎁스를 참조하고 싶으면 reference.Child(string)으로 접근할 수 있습니다.

----------------------------------------
📘 참고: [firebase 공식문서](https://firebase.google.com/docs/database/unity/retrieve-data?hl=ko)
