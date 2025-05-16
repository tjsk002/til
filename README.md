### 자바 성능 튜닝 이야기

#### 1. 디자인 패턴
1. MVC 패턴
   - model 
   - view 
   - controller
2. J2EE 패턴 (java 2 enterprise edition)
   - MVC 패턴의 구조가 기본으로 깔려있다
   - 웹 어플리케이션 서버에서 동작하는 장애복구 및 분산 멀티미어 제공하는 자바 소프트웨어 기능을 추가한 서버를 위한 플랫폼
#### 2. 내가 만든 프로그램의 속도를 알고 싶다
   - 툴을 사용해서 테스트 해볼수있다
          1. APM, 프로파일링 툴 (유료)
          2. 가볍게 테스트 : System 클래스 (무료)
              - System 은 Class이다 static으로 명시되어있다. System 객체를 생성할 수 없다.
              - System.currentTimeMills(현재 시간 리턴)와 System.nanoTime (시간 측정 사용이므로, 오늘 날짜를 알내는 용도로는 사용 못함)
              - JMH(java microbenchmark harness : jdk를 오픈 소스로 제공하는 openJDK에서 만든 라이브러리)
                  - 여러 스레드로 테스트 가능, 굵은 글자로 표현된것만 확인하면 된다
                  - min, avg, max, stdev(표준편차:평균을 중심으로 얼마나 떨어져있는지 값 표현. 수치가 적을수록 평균값이 가깝다는 뜻)
#### 3. 왜 자꾸 String을 쓰지 말라는거야?
1. String은 잘 사용하면 상관이 없지만, 잘못 사용하면 메모리에 영향을 많이준다 (String을 많이 사용하면 GC가 많이 발생한다. 쿼리 생성할때 String 으로 된 문자열을 더하는것이다.) 그 이유는?
   1. 우선 이 얘기를 하기전에, java.lang.Object(클래스 확장)를 제외하고 가장많이 사용하는 객체가 무엇일까?
      - 1위는 String 클래스, 2위 Collection 관련 클래스이다
2. StringBuilder 클래스 StringBuffer 클래스
   - jdk 5버전 기준으로 문자열을 만드는 클래스는 String, StringBuilder, StringBuffer을 가장 많이 사용한다.  그 둘의 차이점은 무엇일까?
       1. StringBuffer 클래스는 안전하게(ThreadSafe) 설계되어있어서 여러개의 스레드에서 하나의 StringBuffer 객체를 처리해도 전혀 문제가 되지 않는다
       2. StringBuilder는 단일 스레드에서만 안전성을 보장 받는다. 그래서 여러개 스레드에서 하나의 StringBuilder를 처리하면 문제가 발생한다
3. 아래 코드를 보면 왜 이렇게 응답시간과 메모리 사용량 차이가 많을까?

```
String strSql = "";
strSql += "select * ";
strSql += "from ";
...
// 응답시간 : 10 회 약 5ms
// 메모리 사용량 : 10회 평균 약 5MB
```
```
StringBuilder strSql = new StringBuilder;
strSql.append("select * ");
strSql.append("from");
...
// 응답시간 : 10회 약 0.3ms
// 메모리 사용량 : 10회 평균 약 371KB
```

- String, StringBuffer, StringBuilder 메모리에서 어떻게 생성되는지 비교해보자
    - String

        | 주소 | String value          |
        | --- |-----------------------|
        | 100 | abcde                 |
        | 150 | abcde + abcde         |
        | 200 | abcde + abcde + abcde |
    
    - StringBuilder, StringBuffer

        | 주소 | String value          |
        | --- |-----------------------|
        | 100 | abcde                 |
        | 100 | abcde + abcde         |
        | 100 | abcde + abcde + abcde |
    - 위에서 보면 ‘String’ 같은 경우에는 주소를 다시 할당 받아서 값을 추가 한 내용을 객체를 다시 생성한다 또 새로운 객체를 생성하고 계속 반복할것이다. 그러면 메모리를 많이 사용하게 되고 응답속도에도 영향을 미치게된다. GC를 하면 할수록 시스템의 CPU를 사용하게 되고 시간도 많이 소요하게 된다.
    - 위에서 보면 StringBuilder, StringBuffer 는 같은 메모리를 사용하여 객체의 크기를 증가시키면서 값을 더한다
    - 뭐 그렇다고 꼭 String이 나쁜것은 아니다 역할에 맞춰 사용하면 된다
        - String : 짧은 문자열 사용할때 사용한다
        - StringBuilder : 스레드에 안전한지 관계 없을때 사용한다
        - StringBuffer: 스레드에 안전하게 사용해야할때. 싱글톤일때 사용 권장 한다
#### 4. 어디에 담아야하는지…
#### 5. 지금까지 사용하던 for 루프를 더 빠르게 할 수 있다고?
#### 6. static 제대로 한번 써보자
#### 7. 클래스 정보, 어떻게 알아낼 수 있나?
#### 8. synchronized는 제대로 알고 써야 한다
#### 9. IO에서 발생하는 병목 현상
#### 10. 로그는 반드시 필요한 내용만 찍자
#### 11. JSP와 서블릿, Spring에서 발생할 수 있는 여러 문제점
#### 12. DB를 사용하면서 발생 가능한 문제점들
#### 13. XML과 JSON도 잘 쓰자
#### 14. 서버를 어떻게 세팅해야할까?
#### 15. 안드로이드 개발하면서 이것만큼은 피하자
#### 16. JVM은 도대체 어떻게 구동될까?
#### 17. 도대체 GC는 언제 발생할까?
#### 18. GC가 어떻게 수행되고 있는지 보고 싶다
#### 19. GC 튜닝을 항상 할 필요는 없다
#### 20. 모니터링 API인 JMX
#### 21. 반드시 튜닝해야 하는 대상은?
#### 22. 어떤 화면이 많이 쓰이는지 알고 싶다
#### 23. 튜닝의 절차는 그때그때 달라요
#### 24. 애플리케이션에서 점검해야 할 사항들