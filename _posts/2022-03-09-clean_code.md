---
layout: post
title:  Clean code - 의미 있는 이름
author: Jihyun
category: Clean code
tags:
- Clean code
- java
date: 2022-03-19 18:14 +0900
---
[로버트 마틴의 Clean Code 정리 글입니다.](https://book.naver.com/bookdb/book_detail.nhn?bid=7390287)



## 1. 의도를 분명히 밝혀라

- 주석이 필요하다면 의도를 분명히 드러내지 못했다는 말

**☹Bad**

```java
int d; // 경과 시간(단위: 날짜)
```

**😊Better**

```java
int daysSinceCreaction;
```



- 코드의 맥락이 코드 자체에 명시적으로 드러나야 한다.

**☹Bad**

```java
public List<int[]> getThem() {
	List<int[]> list1 = new ArrayList<>();
	for (int[] x : theList)
		if(x[0]==4)
			list1.add(x);
	return list1;
}
```

**😊Better**

```java
public List<Cell> getFlaggedCells() {
	List<Cell> flaggedCells = new ArrayList<>();
	for (Cell cell : getBoard)
		if(cell.isFlagged())
			flaggedCells.add(cell);
	return flaggedCells;
}
```



#### ✨실전 적용

**☹Before**

``` JAVA
if(!lesson.getUseFlag().booleanValue()) {
    throw new AccessDeniedException("Deleted lesson!");
}
```

**😊After**

```
if(lesson.isDeleted()) {
    throw new AccessDeniedException("Deleted lesson!");
}
```



## 2. 그릇된 정보를 피하라

- 나름대로 널리 쓰이는 의미가 있는 단어를 다른 의미로 사용하지 말 것
- 서로 흡사한 이름을 사용하지 말 것
- 유사한 개념은 유사한 표기법을 사용할 것
- 숫자와 비슷해보이는 변수로 사용하는 것을 지양할 것 (0, O, 1, l)



## 3. 의미 있게 구분하라

- 연속된 숫자를 덧붙이거나 불용어를 추가하는 방식은 부적절, 이름이 달라야 한다면 의미도 달라져야 한다.

**☹Bad**

``` JAVA
public static void copyChars(char a1[], char a2[]) {
    for(int i=0; i<a1.length; i++) {
        a2[i] = a1[i];
    }
}
```

**😊Better**

```java
public static void copyChars(char source[], char destination[]) {
    for(int i=0; i<source.length; i++) {
        destination[i] = source[i];
    }
}
```

**🤷‍♂️Bad**

```java
getActiveAccount();
getActiveAccounts();
getActiveAccountInfo();
```



## 4. 발음하기 쉬운 이름을 사용하라

- 이상한 줄임말을 만들지 말고 말할 수 있는 이름을 사용하자

**☹Bad**

``` JAVA
class DtaRcrd102 {
    private Date genymdhms;
    private Date modymdhms;
    private final String pszqint = "102";
}
```

**😊Better**

```java
class DtaRcrd102 {
    private Date generationTimestamp;
    private Date modificationTimestamp;
    private final String recordId = "102";
}
```



## 5. 검색하기 쉬운 이름을 사용하라

- 문자 하나를 사용하는 이름과 상수는 텍스트 코드에서 쉽게 눈에 띄지 않는다.
  - e를 변수로 쓴다면 e로 검색 시 오조오억개 나옴
- 긴 이름이 짧은 이름보다 좋다.
- 이름 길이는 범위 크기에 비례해야 한다.
  - 간단한 메서드에서 로컬 변수만 한 문자를 사용한다.
  - 변수나 상수를 코드 여러 곳에서 사용한다면 검색하기 쉽도록 충분한 길이로 선언하는 것이 좋다.

**☹Bad**

``` JAVA
for(int j=0; j<34; j++) {
    s += (t[j]*4)/5;
}
```

**😊Better**

```java
int realDaysPerIdealDay = 4;
const int WORK_DAYS_PER_WEEK = 5;
int sum = 0;
for(int j=0; j<NUMBER_OF_TASKS; j++) {
    int realTaskDays = taskEstimate[j] * realDaysPerIdealDay;
    int realTaskWeeks = realTaskDays / WORK_DAYS_PER_WEEK;
    sum += realTaskWeeks;
}
```



## 6. 인코딩을 피하라

- 헝가리식 표기법
  - 변수 이름에 타입을 작성하지 말 것, 객체는 강한 타입이며 이름에 타입을 넣으면 타입을 바꾸기 어려워진다.
  - 🤦‍♀️PhoneNumber phoneNumberString;
- 멤버 변수 접두어
  - 클래스와 함수는 접두어가 필요 없을 정도로 작아야 한다.
  - 코드를 읽을 수록 접두어와 접미어는 관심 밖으로 밀려난다.
- 인터페이스 클래스와 구현 클래스
  - 내가 다루는 클래스가 인터페이스라는 사실을 굳이 접두어로 알릴 필요가 없다. (ex: IShapeFactory)
  - 굳이 인터페이스와 구현 클래스 중 하나를 인코딩 해야한다면 구현 클래스를 인코딩 한다.
    - ShapeFactory (interface), ShapeFactoryImpl (class)



## 7. 자신의 기억력을 자랑하지 마라

- 코드를 읽으면서 변수 이름을 아는 이름으로 변환해야 한다면 그 변수 이름은 바람직하지 못하다.
- 문자 하나만 사용하는 변수 이름은 문제가 있다. (작은 범위에서 루프에서 반복 횟수를 세는 변수 제외)
- **명료함이 최고**! 남들이 이해하는 코드를 내놓자



## 8. 클래스 이름

- 클래스 이름과 객체 이름은 명사나 명사구가 적합하다.

**👍Good** : Customer, WikiPage, Account, AddressParser

**👎Bad** : 동사, Manager, Processor, Data, Info



## 9. 메서드 이름

- 메서드 이름은 동사나 동사구가 적합하다.
- 접근자, 변경자, 조건자는 javabean  표준에 따라 get, set, is를 붙인다.
- 생성자를 중복정의 할 때는 정적 팩토리 메서드를 사용한다.

**☹Bad**

```java
Complex fulcrmPoint = new Complex(23.0);
```

**😊Better**

```java
Complex fulcrmPoint = Complex.FromRealNumber(23.0);
```



#### ✨실전 적용

**☹Before**

``` java
Lesson lesson = new Lesson(lessonSeq);
```

**😊After**

```java
// version 1
Lesson lesson = Lesson.fromLessonSeq(lessonSeq);
// version 2
Lesson lesson = Lesson.of(lessonSeq);
```



## 10. 기발한 이름은 피하라

- 재미난 이름보다 명료한 이름을 선택하라.
  - *예전 회사에 전설의 안젤리나졸리 브래드피트 함수가 있었다는 후문...*
- 구어나 속어를 사용하지 않는다.



## 11. 한 개념에 한 단어를 사용하라

- 일관성 있는 어휘를 사용한다.
  - 똑같은 메서드를 fetch, retrieve, get으로 제각각 부르면 혼란스럽다.



## 12. 말장난을 하지 마라

- 한 단어를 두 가지 목적으로 사용하지 마라.
  - 예를 들어 add라는 메소드가 기존 값 두개를 더해서 새로운 값을 만드는 맥락이었는데, 기존 집합에 값 하나를 더 추가하는 것을 add로 부르면 기존 메서드와 맥락이 다르다. insert나 append 같은 이름이 적당하다.



## 13. 해법 영역에서 가져온 이름을 사용하라

- 코드를 읽을 사람도 프로그래머라는 사실을 명신한다. 
  - 전산용어, 알고리즘 이름, 패턴 이름, 수학용어를 사용해도 괜찮다.
  - 모든 이름을 Domain 에서 가져올 필요는 없다.
- 기술 개념에는 기술 이름이 가장 적합한 선택이다.



## 14. 문제 영역에서 가져온 이름을 사용하라

- 적절한 프로그래머 용어가 없다면 문제 영역에서 가져온 이름을 사용한다.



## 15. 의미 있는 맥락을 추가하라

- 클래스, 함수, 이름 공간에 넣어 맥락을 부여한다. 
- 모든 방법이 실패하면 마지막 수단으로 접두어를 붙인다.
  - 접두어를 붙인 변수들을 모아 클래스를 생성하면 더 좋다.



## 16. 불필요한 맥락을 없애라

- 모든 클래스 이름에 불필요한 맥락을 넣을 필요는 없다.
- 의미가 분명한 경우 일반적으로는 짧은 이름이 긴 이름보다 좋다.
  - customerAddress, accountAddress는 인스턴스로는 좋은 이름이나 클래스 이름으로는 부적합하다. Address가 적합하다.
  - 포트주소, MAC 주소, 웹주소를 구분해야 한다면 PostalAddress, MAC, URI라는 이름이 괜찮다.



## 마치면서

우리들 대다수는 자신이 짠 클래스 이름과 메서드 이름을 모두 암기하지 못한다. 암기는 도구에 맡기고 우리는 문장이나 문단처럼 읽히는 코드 아니면 적어도 표나 자료 구조처럼 읽히는 코드를 짜는 데만 집중하는데 마땅하다.



#### 출처

> Clean Code p57~73

