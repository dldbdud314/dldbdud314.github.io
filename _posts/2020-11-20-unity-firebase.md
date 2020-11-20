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

FirebaseApp.DefaultInstance.SetEditorDatabaseUrl(해당 주소 붙여넣기);
DatabaseReference reference = FirebaseDatabase.DefaultInstance.GetReference("reference_one");
```

## 1️⃣ 데이터 읽는 방식
일회성으로 읽어 들이는지 지속적으로 읽어 들이는지에 따라 크게 두가지 방식으로 나눌 수 있습니다   
1. 데이터 한번 읽기
해당 방식은 지정된 경로에서 해당 위치의 모든 데이터를 포함하는 정적 스냅샷(DataSnapshot)을 찍습니다. 데이터를 읽어들이면 DataSnapshot의 형태로 이를 받을 수 있습니다.     
📘 [DataSnapshot 참고 자료](https://firebase.google.com/docs/reference/android/com/google/firebase/database/DataSnapshot)

2. 이벤트 수신 대기
이벤트 리스너를 추가하는 방식으로, 데이터의 변화가 있을 때마다 이를 읽어들일 수 있습니다.   
아래와 같은 이벤트들에 리스너를 추가할 수 있습니다.  
![image](https://user-images.githubusercontent.com/57944099/99770279-eedd8000-2b4a-11eb-8ca7-87a9df1c01ef.png)
- 이러한 이벤트는 리스너가 연결될 때 **한 번 호출된 후** 하위 데이터를 포함한 **데이터가 변경될 때마다 다시 호출**된다

## 2️⃣ 데이터 한번 읽기
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
