---
title: "DataSnapshot 중첩된 데이터셋 접근"
categories:
  - firebase
tags:
  - unity
  - firebase
author_profile: true
toc: true
---
## DataSnapshot이란?
- 데이터 읽어들일 때 받아오는 형태(데이터를 담고 있다고 보면 된다)
- 리스너의 파라메터로 전달 (아래 ChildChangedEventArgs args)

```
void HandleChildAdded(object sender, ChildChangedEventArgs args)
    {
        if (args.DatabaseError != null)
        {
            Debug.LogError(args.DatabaseError.Message);
            return;
        }

        //Do something with the data in args.Snapshot
    }
```

### 접근하기
아래와 같은 데이터 구조를 가진 데이터셋이 있다고 하자.
```
reference
ㄴfirstChild
  ㄴfirstGrandChild
    ㄴfirstGrandGrandChild : "abcd"
    ㄴsecGrandGrandChild : "abcd2"
  ㄴsecondGrandChild
    ㄴfirstGrandGrandChild : "efgh"
    ㄴsecGrandGrandChild : "efgh2"
ㄴsecondChild
  ㄴfirstGrandChild
    ㄴfirstGrandGrandChild : "ijk"
    ㄴsecGrandGrandChild : "ijk2"
  ㄴsecondGrandChild 
    ㄴfirstGrandGrandChild : "lmnop"
    ㄴsecGrandGrandChild : "lmnop2"
```
이렇게 중첩된 구조는 Snapshot.Children을 활용하여 아래와 같이 접근하면 좋다.
```
void HandleChildAdded(object sender, ChildChangedEventArgs args)
{
     if (args.DatabaseError != null)
     {
         Debug.LogError(args.DatabaseError.Message);
         return;
     }

     //Do something with the data in args.Snapshot
     Debug.Log("Child: "+ args.Snapshot.Key) //Child
     foreach (DataSnapshot data in args.Snapshot.Children){
       Debug.Log("GrandChild: "+ args.Snapshot.Key) //GrandChild 
       foreach (DataSnapshot data2 in data.Children){
          Debug.Log("GrandGrandChild: "+ args.Snapshot.Key) //GrandGrandChild
          IDictionary grandgrand = (IDictionary)data2.Value;
          Debug.Log(grandgrand["firstGrandGrandChild"]+grandgrand["secondGrandGrandChild"]); //GrandGrandChild의 key값
       }
     }
}
```
