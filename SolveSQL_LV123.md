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

컬럼명 바로 앞에 써주어야 함.

ex. COUNT(DISTINCT order_id)

## strftime 함수
strftime('원하는 형태', timestamp 컬럼명)

timestamp 컬럼의 값들을 원하는 형태(연, 월, 일만 출력 등)로 출력하기 위한 함수.

ex. strftime('%Y-%m-%d', order_purchase_timestamp)

## WITH문
임시 테이블을 만드는 것. 주의할 점은 컬럼이 아닌 테이블이라는 점.

FROM 절에 서브쿼리를 사용하는 것 대신 WITH문으로 임시 테이블을 만들어두고, 메인 쿼리의 FROM절에 그 임시 테이블을 불러와서 사용할 수 있다.

가독성이 좋고 재사용이 가능하다는 장점이 있지만, DB의 효율성 측면에서는 손해일 수도 있다.

이름이 있는 서브쿼리 정도로 이해하면 쉬운데, select절에 서브쿼리를 사용할 수 없으므로 그럴 때 with문을 사용해주면 된다.

ex. 일 별 매출의 합계를 먼저 구하고, 그 일 별 매출의 평균을 구하고 싶을 때.
```SQL
WITH sum_sales as(SELECT sum(total_bill) as day_sales
                  FROM tips
                  GROUP BY day)
SELECT round(avg(day_sales),2) as avg_sales
FROM sum_sales
```

## IN, LIKE
~를 포함하고 있는 값들을 출력하고싶을 때 사용할 수 있음.

- IN: ('apple', 'banana') 형식으로 여러 개의 후보 중 하나의 값과 정확히 일치하는 경우를 찾고싶을 때 사용할 수 있음.

- LIKE: 특정 단어를 포함 하기만 하면 돼서 '%apple%' 등의 형식으로 사용 가능함.

    여러 단어 중 하나를 포함하는 값을 찾고 싶을 때, LIKE에 OR 조건을 사용하고싶다면, OR 다음에 LIKE를 다시 써주어야 함.

    ex. WHERE name LIKE '%Christmas%' OR name LIKE '%Santa%'

## 문자열 출력
다른 언어의 print문 처럼 단순히 문자열을 출력하고 싶을 때, 단순하게 select 다음에 '문자열'을 해주면 끝!

ex. SELECT 'Merry Christmas!'