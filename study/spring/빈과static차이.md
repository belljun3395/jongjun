## Bean과 static 클래스의 차이

최근에 공부를 하면서 Bean과 static 클래스는 어떤 차이가 있을지 문듯 궁금해졌다.

아니 정확히는 어떤 경우에 Bean으로 등록해야하고 어떤 경우에 static 클래스로 만들어도 될지 궁금했다.

특히 @Componet 별다른 고민 없이 붙이고 있다는 생각 역시 들어 정리가 필요하다는 생각이 들었다.



그래서 검색을 해보았는데 역시 나만 고민한게 아니었다! ㅎㅎ

[잘 정리된 글](http://kwon37xi.egloos.com/4844149?fbclid=IwAR0YAxeekXxjB7clzFLpCfqQ90FiwSPTyrN1SH3rG_JFrZ0lvJz_pv_eudI)이 있어 이를 보고 이해한 것을 정리하는 형식으로 글을 남기려고 한다.



해당 글에서 제시하는 **기준은** 다음과 같다.

---

**static 함수 모음 클래스의 모든 함수는 인자가 동일할 경우 항상 동일한 결과를 리턴해야 한다. **

**이 규칙을 지킬 수 없으면 POJO Bean으로 만들라.**

**즉 함수 안에서는 외부 자원에 대해 하나도 의존하면 안된다.** 

**외부 자원은 그 실행 결과의 일관성을 보장할 수 없기 때문이다.**

---



해당 글에서는 StringUtils와 EmailUtils를 가지고 예를 든다.

StringUtils를 생각해보면 인자가 동일한 경우 항상 동일한 결과를 리턴한다.

StringUtils.contains("hello world", "hello")는 <u>항상 true라는 결과를 보장한다.</u>



하지만 EmailUtils를 생각해보자.

EmailUtils는 SMTP라는 외부 자원에 의존한다.

SMTP가 문제가 없다면 EmailUtils도 StringUtils처럼 인자가 동일한 경우 동일한 결과를 리턴한다.

<u>하지만 SMTP가 항상 문제가 없을 것이라는 확신을 할 수 있을까?</u>

그렇지 못할 것이다.



그리고 글에서는 static 함수 모음을 만드는 방법을 소개한다.

우선 이미 구현된 것을 **상속** 받아 구현하라는 것이다.

생각해보면 지인이 알려준 현업이 작성하였다고한 코드에도 위의 방법이 적용되어 있다.

```java
@Getter
public class ApiResponse<B> extends ResponseEntity<B> implements Serializable {

	public ApiResponse(final HttpStatus status) {
		super(status);
	}
  ...
}

```



