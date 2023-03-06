---
title: "[Good Code, Bad Code]1.코드 계약"
excerpt: "코드 계약과 세부 조항"

categories:
  - Good Code, Bad Code
tags:
  - Good Code, Bad Code
  - 좋은 코드, 나쁜 코드
  - 코드 계약
last_modified_at: 2023-02-22
toc: true
toc_sticky: true
---

개발자들이 다른 사람이 작성한 코드와 상호작용할 때, 코드의 중요한 사항과 코드의 사용법을 다른 개발자에게 전달할 수 있어야 한다. 

전달하는 방법엔 여러가지가 있는데, 이번 포스트에서는 더 신뢰할 수 있는 기법들에 대해 소개한다.  

이는 [고품질 코드의 필수 요소](https://solesie.github.io/good%20code,%20bad%20code/Good-Code,-Bad-Code-0.%EA%B3%A0%ED%92%88%EC%A7%88-%EC%BD%94%EB%93%9C/#2%EA%B3%A0%ED%92%88%EC%A7%88-%EC%BD%94%EB%93%9C%EB%A5%BC-%EB%8B%AC%EC%84%B1%ED%95%98%EA%B8%B0-%EC%9C%84%ED%95%B4)의 2번과 3번, 즉 "예측 가능한 코드", "오용하기 어려운 코드"
와 관련이 있다.   
<br>
<br>

# 1.코드 계약

<I>계약에 의한 프로그래밍 철학</I> 은 서로 다른 코드 간의 상호작용을 마치 계약처럼 생각한다. 

어떤 코드를 호출하는 코드는 <strong>선결 조건</strong>을 충족해야 하며, 호출된 코드는 <strong>사후 조건</strong>에 맞게 반환하거나 일부 상태를 수정한다.  

모든 것이 이 계약에서 정의되기 때문에 예상과 다르게 벗어나는 일이 없다. 

코드 계약은 아래와 같은 세 가지 범주로 나눌 수 있다.  
- 선결 조건: 코드를 호출하기 전에 사실이어야 하는 것(e.g. 인자)
- 사후 조건: 코드가 호출된 후에 사실이어야 하는 것(e.g. 반환값)
- 불변 사항: 코드가 호출되기 전 후로 변경되지 않아야하는 사항   

그리고 계약은 늘 그렇듯 명확한 부분과 세부 조항이 있다.  

![세부 조항](/assets/images/%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202023-02-23%20%EC%98%A4%ED%9B%84%2012.27.45.png){: width="40%" height="40%" .align-center}  

회원가입을 한다가 명확한 부분이고 긴 스크롤을 내려야만 하는 약관 동의는 세부 조항이라 할 수 있다. 

## 1) 코드 계약의 명확한 부분

- 함수와 클래스 이름
- 인자 유형
- 반환 유형
- 검사 예외

코드 계약의 명확한 부분에는 이와 같은 요소들이 있다. 네 가지 요소 모두 틀릴 경우 <strong>컴파일 조차 되지 않는다.</strong> 

## 2) 코드 계약의 세부 조항

- 주석문과 문서
- 비검사 예외     

다른 개발자가 유의깊게 보지 않는 경우가 매우 많고 정확하지 않을 수도 있으므로 조건을 명확하게 하는게 베스트다.  
<br>
<br>

# 2. 세부 조항을 제거하는 방법

세부 조항에 너무 의존하는 코드들이 존재한다.  

이는 주석문에 관해 포스트를 작성할 때도 언급할 것이다.  

```kotlin
class UserSettings{
  UserSettings(){ ... }

  //이 함수를 사용해 설정이 올바르게 로드되기 전까지는 다른 어떤 함수도
  //호출해서는 안된다.
  //설정이 성공적으로 로드되면 참을 반환한다.
  Boolean loadSettings(File location){ ... }

  //init()은 다른 함수 호출 이전에 호출해야 하지만
  //loadSettings() 함수 호출 이후에만 호출해야 한다.
  void init(){ ... }

  //사용자가 선택한 UI의 색상을 반환한다.
  //선택된 색상이 없거나, 설정이 로드되지 않았거나, 초기화되지 않은 상태면
  //널을 반환한다.
  Color? getUiColor(){ ... }
}
```

위 코드의 세부 조항은 너무 많다.  

일단 `loadSettings()` -> `init()` 호출 후에 클래스를 사용가능하고, `loadSettings()`가 `false`를 반환한다면 다른 함수를 호출해선 안된다.   

게다가 `getUiColor()`이 `null`을 반환하는 것은 선택된 색상이 없거나 클래스가 로드되지 않은 상태를 의미한다.   

이는 또한 예측 가능한 코드도 아니다.  

대부분의 사용자는 이 함수를 호출할 때 `null`값이 반환된다면 사용자가 색상을 선택하지 않았다 정도로 생각할 것이다. 

즉, 세부 조항을 자세히 읽지 않는다면 코드를 오용할 가능성이 매우 높아진다. 

<strong>
다른 개발자가 코드를 올바르게 사용하기 위해 세부 조항에 의존하기보다, 잘못된 일을 하는 것을 처음부터 불가능하게 하는 것이 좋다.   
</strong>

<strong>
예를들어 오용되면 컴파일조차 되지 않도록 하는 것이 목표다.      
</strong>

이에 맞게 변경해보자. 

```kotlin
class UserSettings{
  private UserSettings(){ ... }

  static UserSettings? create(File location){
    UserSettings settings = new UserSettings();
    if(!settings.loadsettings(location)){
      return null;
    }
    settings.init();
    return settings;
  }

  private Boolean loadSettings(File location){ ... }

  private void init(){ ... }

  //사용자가 선택한 UI의 색상을 반환한다.
  //선택된 색상이 없으면 널을 반환한다.
  Color? getUiColor(){ ... }
}
```

`create()` 라는 <strong>정적 팩토리 함수</strong>가 추가된다.  

`create()` 정적 팩토리 함수 사용을 강제하기 위해서 생성자는 프라이빗으로 설정한다.  

`UserSettings` 클래스는 이 정적 팩토리 함수를 사용해 초기화가 완전히 이루어진 인스턴스를 얻는 것만 가능하다.  

`getUiColor()` 함수가 널값을 반환한다면 이는 이제 선택한 색상이 없다는 의미만 가진다. 