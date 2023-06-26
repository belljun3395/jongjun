## Iterator



반복자는 컬렉션의 요소들의 **기본 표현을 노출하지 않고 그들을 하나씩 순회할 수 있도록** 하는 행동 디자인 패턴이다.

*출처: [Guru 반복자 패턴](https://refactoring.guru/ko/design-patterns/iterator)*



![img](https://upload.wikimedia.org/wikipedia/commons/thumb/1/13/Iterator_UML_class_diagram.svg/500px-Iterator_UML_class_diagram.svg.png)

*출처: https://ko.wikipedia.org/wiki/%EB%B0%98%EB%B3%B5%EC%9E%90_%ED%8C%A8%ED%84%B4*

 

이번에는 코드로 확인해 보자.

```java
@Getter
public class Books {

    private List<Book> source;
}

public class Book {

}
```

우선 Book을 가지고 있는 Books라는 일급 컬렉션이 있다.



반복자 패턴을 적용하지 않는다면 아래와 같이 Books를 순회해야 했을 것이다.

```java
Books books = new Books();
List<Book> source = books.getSource();
for(Book book : source) {
    // doSomething..
}
```

이렇게 되면 Books가 가지고 있는 **source의 타입이 클라이언트 코드에 노출**된다.

이는 Books의 source 타입이 변경된다면 그 영향이 클라이언트 코드까지 미친다는 것을 의미한다.



하지만 Java에 이미 존재하는 Iterator 인터페이스를 사용하여 BookIterator를 만들어 사용한다면 어떨까?

우선 BookIterator 부터 살펴보자.

```java
public class BookIterator implements Iterator<Books> {

    private Iterator<Book> bookIterator;

    public BookIterator(Books books) {
        this.bookIterator = books.getSource().iterator();
    }

    @Override
    public boolean hasNext() {
        return false;
    }

    @Override
    public Books next() {
        return null;
    }
}
```

Books의 source가 이미 iterator을 가지고 있기에 해당 iterator를 bookIterator로 선언 해주었다.



이렇게 BookIterator를 구현한다면 아래와 같이 Books를 순회할 수 있다.

```java
Books books = new Books();
BookIterator bookIterator = new BookIterator(books);
while(bookIterator.hasNext()) {
    Books next = bookIterator.next();
    // doSomething
}
```

반복자 패턴을 사용하지 않았을 때와 비교하면 **Books의 source 타입이 직접적으로 클라이언트 코드에 노출되지 않고 BookIterator만 노출**되는 것을 확인할 수 있다.

이렇게 클라이언트 코드에 source가 노출되지 않는다면 source의 타입을 변경하여도 클라이언트 코드는 그것에 관심을 가지지 않을 수 있다는 장점을 가진다.



이러한 반복자 메서드 패턴은 다음과 같은 경우에 적용할 수 있다고 한다.

1. 컬렉션이 내부에 복잡한 데이터 구조가 있지만 **이 복잡성을 보안이나 편의상의 이유로 클라이언트들로부터 숨기고 싶을 때**

반복자는 복잡한 데이터 구조와 작업 시의 세부 사항을 캡슐화하여 클라이언트에 컬렉션 요소들에 접근할 수 있는 몇 가지 간단한 메서드들을 제공한다.

이 접근 방식은 클라이언트에게 매우 편리하다.

또 클라이언트가 컬렉션과 직접 작동할 때 클라이언트가 수행할 수 있는 부주의하거나 악의적인 행동들로부터 컬렉션을 보호한다.



2. 앱 전체에서 순회 코드의 중복을 줄이고 싶을 때



3. 반복자 패턴은 코드가 다른 데이터 구조들을 순회할 수 있기를 원할 때 또는 **이러한 구조들의 유형을 미리 알 수 없을 때** 사용한다.

컬렉션들과 반복자들 모두에 몇 개의 일반 인터페이스를 제공한다.

이러한 인터페이스들은 그들이 구현하는 다양한 컬렉션들 및 반복자들을 전달받아도 여전히 작동한다.