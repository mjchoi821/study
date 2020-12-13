## Item 8. finalizer와 cleaner 사용을 피하라



finalizer 와 cleaner 는 자바의 객체 소멸자이나, 직접적인 사용을 권장하지 않는다.



// TODO: 잘 사용되지 않는 부분이라 추후 정리할 예정이다.



cleaner는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자.
언제 호출될지 보장할 수 없다.

