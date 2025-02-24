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

### cf.

SQLite3 환경에서 **`BETWEEN` 대신 `WHERE '2021-01-01' <= strftime('%Y-%m-%d', rent_at) <= '2021-01-31'`을 사용하면 조건이 제대로 동작하지 않을 가능성이 있습니다.**  

그 이유는 **SQLite에서 `<=` 연산자의 평가 방식 때문**입니다.

---

## **🚨 핵심 문제: SQLite에서 `A <= B <= C`가 올바르게 평가되지 않을 가능성**
```sql
WHERE '2021-01-01' <= strftime('%Y-%m-%d', rent_at) <= '2021-01-31'
```
이렇게 작성하면 **SQL에서 예상한 대로 동작하지 않을 수 있습니다.**  
SQL에서는 `A <= B <= C`와 같은 삼중 비교 연산을 직접 평가하지 않습니다.  
대신 **왼쪽부터 차례대로 평가되는데, 예상과 다르게 결과가 나올 수도 있습니다.**

### **📌 잘못된 평가 방식**
```sql
('2021-01-01' <= strftime('%Y-%m-%d', rent_at)) <= '2021-01-31'
```
이렇게 **왼쪽부터 평가**되는데,  
1. `'2021-01-01' <= strftime('%Y-%m-%d', rent_at)` → **이 결과는 `TRUE` 또는 `FALSE`**
2. `TRUE <= '2021-01-31'` → 여기서 문제 발생 ⚠️  
   - `TRUE`는 `1`, `FALSE`는 `0`으로 해석될 수 있음
   - 즉, `1 <= '2021-01-31'` 같은 비교가 이루어지는데, 문자열과 숫자 비교가 비정상적으로 처리될 가능성이 있음

결과적으로 **예상한 대로 필터링되지 않을 가능성이 높습니다.**

---

## **✅ 해결 방법: `AND` 연산자를 사용한 올바른 조건**
SQLite에서는 **두 개의 별도 비교 조건을 `AND`로 묶어서 명확하게 표현**해야 합니다.
```sql
WHERE '2021-01-01' <= strftime('%Y-%m-%d', rent_at)
  AND strftime('%Y-%m-%d', rent_at) <= '2021-01-31'
```
이렇게 하면 SQLite가 올바르게 평가하며, **날짜 비교가 정상적으로 작동합니다.**  

---

## **🚀 결론**
### **❌ 이렇게 하면 안 됨**
```sql
WHERE '2021-01-01' <= strftime('%Y-%m-%d', rent_at) <= '2021-01-31'
```
✔ **이렇게 해야 SQLite에서 정확히 동작함**
```sql
WHERE '2021-01-01' <= strftime('%Y-%m-%d', rent_at)
  AND strftime('%Y-%m-%d', rent_at) <= '2021-01-31'
```
또는 **BETWEEN을 활용하는 것도 가능**:
```sql
WHERE strftime('%Y-%m-%d', rent_at) BETWEEN '2021-01-01' AND '2021-01-31'
```

즉, **SQLite에서는 삼중 비교(`<=` 연산자 두 개 연속 사용)를 피하고, `AND`로 명확하게 분리해야 합니다!** 🚀

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