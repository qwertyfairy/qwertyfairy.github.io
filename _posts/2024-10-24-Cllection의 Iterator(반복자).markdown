---
layout: post
categories: [Java, 학습]
title: "Cllection의 Iterator(반복자)"
---
<span style="font-size: 20px;">**Iterator, 반복자**</span>

Collection은 Iterator(반복자)를 상속하고있다. 따라서 Collection을 상속한 클래스들은 Iterator를 반환받을 수 있다. 이 Iterator의 역할은 저장된 요소들을 순차적으로 참조할때 사용한다. 

```java
/** Collection<E>의 iterator를 반환하는 메서드*/
Iterator<E> iterator()
```

<br>
**💡Iterator의 기본 메서드들**
- E next()  : 다음 인스턴스의 참조값을 반환
- boolean hasNext() : next호출시 참조 값 반환 가능 여부 확인
- remove() : next호출을 통해 반환했던 인스턴스 삭제

<br>

Iterator는 next()를 호출할때 마다 첫번째 요소를시작으로 다음 요소의 참조값을 차례로 반환한다. 더이상 반환할 대상이 없을때 NoSuchElementException예외를 발생시킨다. 이를 방지하고자 다음과 같이 반복문을 구성해야한다.

```java
while(itr.hasNext()){ // 반환할 대상이 있는지 체크
	str = itr.next(); 
}
```

<br>
for-each문과 다르게 interator를 이용하면 **반복 중간에 특정 요소를 삭제하는것이 가능**하다.
iterator는 생성과 동시에 첫번째 요소를 가리키고 next()가 호출될때 다음 요소로 이동한다. 다시 첫번째 요소를 가리키게하려면 iterator를 다시 얻어야한다.

```java
Iterarot<String> itr = list.iterator();
```

<br>
<span style="font-size: 20px;">**ListIterator, 양방향 반복자**</span>

List<E>를 구현하는 클래스의 인스턴스들만 얻을수 있는 ListIterator(양방향 반복자) 라는것이 있다.

```java
//List의 다음 메서드로 얻을수 있다, ListIterator는 Iterator를 상속한다.
public ListIterator<E> listIterator() 
```
<br>
배열이나 연결리스트와 같은 자료 구조의 특성으로 양쪽 방향으로 이동이 가능하다는 특징이 있다.
<br>

**💡ListIterator의 대표 메서드들**
- E next()  : 다음 인스턴스의 참조값을 반환
- boolean hasNext() : next호출시 참조 값 반환 가능 여부 확인
- remove() : next호출을 통해 반환했던 인스턴스 삭제
- E previous() : next와 동일하나 방향만 반대
- boolean hasPrevious() : hasNext와 동일하나 방향만 반대
- void add(E e) : 추가
- void set(E e) : 변경

<br>

덧붙여서 **add()는 다음과 같은 특징**을 지닌다. 특히 3번을 모르고 사용하면 예상과는 전혀 다른 결과가 나올수 있기때문에 반드시 기억하고 있어야 한다.

1. next호출 후 호출하면 반환된 인스턴스 뒤에 삽입된다.
2. previous호출 후 호출하면 반환된 인스턴스 앞에 삽입된다.
3. next에는 영향 받지않으나, previous에는 영향을 받는다.

```java
  List<String> list = Arrays.asList("Toy","Box","Robot","Box");
        list = new ArrayList<String>(list);

        ListIterator<String> litr = list.listIterator();

        String str;
        while (litr.hasNext()){
            str = litr.next(); 
            System.out.print(str + '\t');
            if (str.equals("Toy")){
                litr.add("Toy2"); 
            }
        }

        System.out.println();

        while (litr.hasPrevious()){
            str = litr.previous();
            System.out.print(str + '\t');
            if (str.equals("Robot")){
                litr.add("Robot2");
            }
        }

        System.out.println();

        for (Iterator<String> itr = list.iterator(); itr.hasNext();){
            System.out.print(itr.next() + '\t');
        }
        System.out.println();
        
        
        /**
        ~ 결과 값 ~ 
        Toy Box Robot Box
        Box Robot Robot2 Box Toy2 Toy
        Toy Toy2 Box Robot2 Robot Box
        */
```
<br>
참고로 for-each문은 컴파일 과정에서 다음과 같이 수정된다.

```java
for(String s: list){
	System.out.print( s + '\t');
}

//컴파일 과정에서 변경된 for-each문
for(Iterator<String> itr = list.iterator(); itr.hasNext(); ){
		System.out.print( itr.next() + '\t');
}
```