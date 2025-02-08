# SQLITE 문법 정리

## 표본분산 구하기
VAR_SAMP(x)

## 대문자 출력하기
upper(X)

## m번째부터 n번째까지만 출력하기
substr(문자열, m, n)

ex. substr('apple', 2, 4)이면 ppl이 출력됨.

특정 문자열 대신 컬럼명 삽입도 가능.

## group by 이후 그룹 별 조건 걸기
having절 이용하기.

## DISTINCT 함수
중복값을 제거하고 출력해주는 함수. 

DISTINCT 뒤에 괄호 없이 컬럼명을 바로 써준다.

ex. DISTINCT order_id