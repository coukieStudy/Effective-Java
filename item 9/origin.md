# try-finally 보다는 try-with-resources 를 사용하라

대상이 되는 자원들: close() method를 호출해야 하는 자원들 (ex. InputStream, OutputStream, java.sql.Connection)

* 기존의 try-finally code
```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
기존 방식의 단점
1. 자원이 둘 이상이면 try-finally 방식은 가독성이 지나치게 떨어진다. 생성한 모든 자원에 대해서 close()를 호출해야 하기 때문에 여러 try-finally가 중첩적으로 생기게 된다.
2. try와 finally 블록 모두에서 예외가 생긴다면, 코드 작성자가 원인을 알고 싶은 예외 (try 문 안에서의 Exception)에 관한 정보가 남지 않게 되어 디버깅이 복잡해진다. 

위의 문제점을 해결할 수 있는 방식이 try-with-resources 이다. 이 구조를 사용하려면 해당 자원이 AutoClosable 인터페이스를 구현해야 하는데, 이 인터페이스는 void close() 메서드만 정의하면 된다.

* try-with-resources 를 적용한 코드
```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
1. 여러 자원에 대해서 try ( ... )로 선언해주기만 하면 별도로 close() 를 호출하지 않아도 된다.
2. try와 close 모두에서 예외가 생긴다면,close()에서 발생하는 예외는 숨겨지고(suppressed), try에서 발생한 예외가 기록된다 - javadoc : https://docs.oracle.com/javase/tutorial/essential/exceptions/tryResourceClose.html

Harry 님의 질문 - close()에서 발생한 예외가 숨겨지고 try에서의 예외가 발생하는 원리는 무엇인가요?

```java
class TryException extends RuntimeException {
}

class CloseException extends RuntimeException {
}

class CloseError implements AutoCloseable{
	public void close() {
		throw new CloseException();
	}
	public void throwsException(boolean b) {
		if(b) throw new TryException();
	}
}
```
기존의 try-catch
```java
public static void main(String[] args) {
	CloseError closeError = new CloseError();
	try {
		closeError.throwsException(false);
	} finally {
		closeError.close();
	}
}
```
결과: throwsException(false), throwsException(true) 모두 CloseException이 발생

try-with-resources
```java
public static void main(String[] args) {
	try (CloseError closeError = new CloseError()){
		closeError.throwsException(true);
	}
}
```
결과: throwsException(false) 이면 CloseException 발생, throwsException(true) 이면 TryException 발생(TryException에 CloseException이 suppress된 형태)

* 기존의 try-finally 에서 자원들은 그대로 둔 채, try-finally 문에 추가를 하는 방식으로 try-with-resources와 동일한 결과를 낼 수 있을까?
Throwable.addSuppress 활용하면 가능할 수 있다.
```java
public static void main(String[] args) {
	CloseError closeError = new CloseError();
	try {
		closeError.throwsException(false);
	} catch (Exception exception_in_try_statement) {
		try {
			closeError.close();
		} catch (Exception exception_in_close_statement) {
			exception_in_try_statement.addSuppressed(exception_in_close_statement);
		}
		throw exception_in_try_statement;
	}
	closeError.close();
}
```
결과 : try-with-resources 와 동일
