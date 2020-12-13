## Item 9. try-finally보다는 try-with-resources를 사용하라



InputStream, OutputStream, java.sql.Connection 등의 자원은 사용 후 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.
하지만 이는 개발자가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다. 이런 자원 중 상당수가 안전망으로 finalizer를 활용하고는 있지만 finalizer는 그리 믿을만하지 못하다(아이템 8).



전통적인 수단으로 자원을 닫는 동작을 보장하는 수단으로 try-finally 가 쓰였다.
하지만 자바7에서 try-with-resources 가 추가되며 많은 이점이 생겼다.
코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용해졌다.
아래의 비교 코드를 통해 장점을 이해하고 앞으로 try-with-resources 를 사용하도록 하자.



- close 해야 할 자원이 둘 이상일 때, try-finally 방식

```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    } finally{
      out.close();
    }
  } finally {
    in.close();
  }
}
```

코드가 너무 복잡하다🙃



- 복수의 자원을 처리하는 try-with-resources 방식

```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst)) { // try에 ()로 초기화 구문이 들어간다는 부분이 조금 생소하지만, 익숙해지자
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  }
}
```

코드길이도 훨씬 짧을 뿐더러 가독성도 좋아졌다.

추가적인 장점으로 기존 try-finally 방식에서는 try 와 finally 양쪽에서 예외가 발생하면, finally 쪽의 예외는 숨겨지고 try 에서 발생한 예외가 기록되지만, try-with-resources 방식은 스택 추적 내역에 suppressed 라고 숨겨졌다는 꼬리표를 달고 출력된다.
또 try-with-resources 방식에서도 catch 절을 쓸 수 있다.

이로써 우리는 try-with-resources 를 사용하지 않을 이유가 하나도 없다는것을 알 수 있게 됐다. 앞으로 try-with-resources 를 사용하도록 하자.