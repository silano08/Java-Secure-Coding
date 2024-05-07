SQL 삽입(Injection)
====

## SQL 삽입

SQL 삽입이란? 프로그램에 SQL을 삽입하여 내부 DB 서버의 데이터를 유출 및 변조 혹은 관리자 인증을 우회하는 보안 약점

### 해결 방법
동적 쿼리에 사용되는 입력 데이터에 예약어 및 특수문자가 입력되지 않게 필터링 되도록 설정하여 방지할 수 있음 (*동적쿼리 = 질의어 코드를 문자열 변수에 넣어 조건에 따라 질의를 동적으로 변경하여 처리하는 방식)

### 실제 사례


(출처 : https://www.devkuma.com/docs/secure-coding-guide/sql-injection/#google_vignette)

### 예방 방법?

jpa 사용하기

왜?