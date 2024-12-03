# 1. PIVOT과 UNPIVOT

## a. PIVOT과 UNPIVOT절 개념 정리 

SQL에서 `PIVOT`과 `UNPIVOT` 절은 데이터의 구조를 다른 형식으로 변환하는 데에 유용하게 사용된다.

 `PIVOT`은 행 데이터를 열로 변환하고, `UNPIVOT`은 열 데이터를 행으로 변환하는 데 사용됨. 두 절 모두 주로 데이터를 집계하거나 특정 형식으로 변환할 때 유용함.

### 1. `PIVOT`
`PIVOT`은 행 데이터를 열로 변환하는 것. 주로 데이터 분석에서 여러 개의 값들을 한 열로 나열할 때 사용되는데, 예를 들어, 월별 매출 데이터를 연도별로 나열하는 경우에 사용됨.

**기본 문법:**
```sql
SELECT <고정 컬럼들>, [새로운_열1], [새로운_열2], ...
FROM (
    SELECT <고정 컬럼들>, <행을 기준으로 변환할 컬럼>, <값으로 변환할 컬럼>
    FROM <테이블>
) AS source
PIVOT (
    <집계 함수>(<값으로 변환할 컬럼>)
    FOR <행을 기준으로 변환할 컬럼> IN ([새로운_열1], [새로운_열2], ...)
) AS pivot_table;
```

**예시:**
```sql
SELECT Year, [Q1], [Q2], [Q3], [Q4]
FROM (
    SELECT Year, Quarter, Sales
    FROM SalesData
) AS source
PIVOT (
    SUM(Sales)
    FOR Quarter IN ([Q1], [Q2], [Q3], [Q4])
) AS pivot_table;
```
이 예시는 `SalesData` 테이블에서 `Year`별로 각 분기(`Q1`, `Q2`, `Q3`, `Q4`)의 `Sales` 합계를 보여주는 것임.

### 2. `UNPIVOT`
`UNPIVOT`은 열 데이터를 행으로 변환하는 것. 즉, 여러 열을 하나의 열로 변환하여 각 열의 값을 행 형태로 나열할 수 있움.

**기본 문법:**
```sql
SELECT <고정 컬럼들>, <새로운_행 값>, <새로운_컬럼 이름>
FROM (
    SELECT <고정 컬럼들>, [새로운_열1], [새로운_열2], ...
    FROM <테이블>
) AS source
UNPIVOT (
    <새로운_행 값>
    FOR <새로운_컬럼 이름> IN ([새로운_열1], [새로운_열2], ...)
) AS unpivot_table;
```

**예시:**
```sql
SELECT Product, Quarter, Sales
FROM (
    SELECT Product, Q1, Q2, Q3, Q4
    FROM SalesData
) AS source
UNPIVOT (
    Sales FOR Quarter IN (Q1, Q2, Q3, Q4)
) AS unpivot_table;
```
이 예시는 `SalesData` 테이블에서 `Q1`, `Q2`, `Q3`, `Q4` 열을 `Quarter`라는 새로운 열로 변환하고, 각 분기의 `Sales` 데이터를 행으로 나열하는 것임.

### 차이점:
- **PIVOT**: 행 데이터를 열로 변환 (집계 함수 사용)
- **UNPIVOT**: 열 데이터를 행으로 변환


## b. HackerRank 문제 풀이

**문제 설명**
각 이름이 알파벳순으로 정렬되고 해당 직업 아래에 표시될 수 있도록 직업의 직업 열을 pivot 해야하는 문제. 

출력 열 헤더는 각각 의사, 교수, 가수 및 배우여야 하며, 직업에 해당하는 이름이 더 이상 없을 때 NULL을 출력해야 함.

직업의 value unique 값은  Doctor, Professor, Singer, Actor이다.

### 정답 코드
```
SET @D=0, @P=0, @S=0, @A=0;
SELECT MIN(Doctor), MIN(Professor), MIN(Singer), MIN(Actor)
FROM (SELECT CASE WHEN Occupation = 'Dpctor' THEN Name END AS Doctor,
             CASE WHEN Occupation = 'Professor' THEN Name END AS Professor,
             CASE WHEN Occupation = 'Singer' THEN Name END AS Singer,
             CASE WHEN Occupation = 'Actor' THEN Name END AS Actor,
        CASE WHEN Occupation = 'Doctor' THEN(@D:=@D+1)
             WHEN Occupation = 'Doctor' THEN(@D:=@D+1)
             WHEN Occupation = 'Doctor' THEN(@D:=@D+1)
             WHEN Occupation = 'Doctor' THEN(@D:=@D+1)
             END AS RowNumber 
      FROM Occupations 
      ORDER BY Name) sub 
GROUP BY RowNumber
```

# 2. 성능 최적화 기법

## a. SQL 쿼리 성능 최적화를 위한 튜닝 팁 칼럼 요약

SQL 쿼리 성능 최적화는 데이터베이스의 효율성을 높여 응답 속도를 빠르게 하고, 자원 낭비를 줄이며, 시스템 장애를 예방하는 중요한 작업이다.

### 1. **좌변을 연산하지 마라**
   - **문제**: `YEAR(date) = 2021`처럼 좌변에서 연산을 하면, 데이터베이스가 인덱스를 제대로 활용하지 못한다. 이는 성능 저하를 초래할 수 있다.
   - **해결책**: 연산을 우변으로 이동하여 인덱스를 효율적으로 활용할 것.
   - **예시**:
     ```sql
     SELECT * FROM sales 
     WHERE date >= '2021-01-01' AND date <= '2021-12-31';
     ```
     이렇게 하면 `date` 컬럼에 대한 인덱스를 활용하여 성능을 최적화할 수 있다.

### 2. **OR 대신 UNION을 사용하라**
   - **문제**: `OR`을 사용하면 데이터베이스가 여러 조건을 동시에 검색해야 하므로 인덱스를 제대로 활용할 수 없다.
   - **해결책**: `OR` 대신 `UNION`을 사용하여 조건별로 별도의 쿼리를 실행하고 결과를 합친다. 이렇게 하면 각 쿼리에서 인덱스를 효율적으로 활용할 수 있다.
   - **예시**:
     ```sql
     SELECT * FROM employees WHERE department = 'Marketing'
     UNION
     SELECT * FROM employees WHERE department = 'IT';
     ```

### 3. **필요한 Row와 Column만 선택하여 성능 최적화하기**
   - **문제**: 불필요한 컬럼과 행을 가져오는 것은 성능 저하의 주범이다.
   - **해결책**: 필요한 데이터만 선택하고, 서브쿼리나 조인에서 불필요한 컬럼을 제외하여 최적화 할 것.
   - **예시**:
     ```sql
     SELECT name, email 
     FROM employees 
     WHERE department = 'Marketing' AND sales >= 100000;
     ```
     이렇게 하면 필요한 정보만 가져오게 되어 성능이 향상된다.

### 4. **분석 함수를 활용하여 쿼리 성능 높이기**
   - **문제**: 복잡한 집계 작업을 전통적인 방법으로 처리하면 성능이 떨어질 수 있다.
   - **해결책**: SQL 분석 함수(예: `ROW_NUMBER()`, `RANK()`, `LEAD()`, `LAG()`)를 사용하여 성능을 향상시킬 수 있다. 분석 함수는 데이터를 그룹화하지 않고 각 행에 대해 연산을 수행할 수 있게 해준다.
   - **예시**:
     ```sql
     SELECT 
       name, 
       department, 
       salary,
       ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
     FROM employees;
     ```

### 5. **와일드카드(%)는 끝에 작성하는 것이 더 좋다**
   - **문제**: `LIKE '%term'`처럼 와일드카드를 앞에 쓰면 인덱스를 활용할 수 없다. 이는 성능에 큰 영향을 미친다.
   - **해결책**: 와일드카드를 끝에 배치하여 인덱스를 활용할 것.
   - **예시**:
     ```sql
     SELECT * FROM products WHERE name LIKE 'term%';
     ```
     이 방식이 성능을 더 최적화한다.

### 6. **계산값을 미리 저장해두고, 나중에 조회할 것**
   - **문제**: 쿼리에서 반복적으로 계산을 수행하면 자원을 낭비하게 된다.
   - **해결책**: 계산된 값을 미리 저장해두고 필요할 때 조회하는 방법을 활용할 것.
   - **예시**: 계산된 값을 테이블에 저장하거나, 미리 계산된 데이터를 별도의 뷰나 테이블로 저장하여 조회하는 방식이 좋다.

### 추가로 알아두어야 할 기본 지식: **인덱싱**
   - 인덱싱은 데이터베이스에서 데이터를 빠르게 찾을 수 있게 돕는 구조이다. 인덱스가 있으면, 데이터를 스캔하는 대신 빠르게 특정 데이터를 찾을 수 있다. 인덱스를 잘 활용하는 것이 성능 최적화의 핵심이다.

이 팁들은 특히 대규모 데이터를 처리하는 데 유용하며, 쿼리 성능을 개선하는 데 큰 도움이 된다.


## b. 문제 풀이

**[문제]**

여러분은 `customer_orders`라는 테이블을 관리하는 데이터베이스 관리자로 일하고 있습니다. 이 테이블에는 고객의 주문 정보가 저장되어 있으며, 각 고객이 주문한 제품과 수량, 가격 정보가 포함되어 있습니다. 또한, 고객들이 특정 제품을 재구매한 비율을 계산하려고 합니다.

### 테이블 구조:

1. **customers** 테이블
    - `customer_id` (고객 ID, PRIMARY KEY)
    - `name` (고객 이름)
2. **orders** 테이블
    - `order_id` (주문 ID, PRIMARY KEY)
    - `customer_id` (고객 ID, FOREIGN KEY)
    - `order_date` (주문 날짜)
3. **order_details** 테이블
    - `order_id` (주문 ID, FOREIGN KEY)
    - `product_id` (제품 ID)
    - `quantity` (수량)
    - `unit_price` (단가)

---

### 요구 사항:

1. **`avg_order_value`**: 고객별 평균 주문 금액을 계산하여 `customers` 테이블에 업데이트하세요.
    - `avg_order_value`는 각 고객이 한 번의 주문에서 지출한 평균 금액입니다.
2. **`total_spent`**: 고객별 총 지출 금액을 계산하여 `customers` 테이블에 업데이트하세요.
    - `total_spent`는 고객이 지금까지 지출한 총 금액입니다.
3. **`num_orders`**: 고객이 총 몇 번의 주문을 했는지 계산하여 `customers` 테이블에 업데이트하세요.
    - `num_orders`는 고객이 주문한 총 개수입니다.
4. **`repurchase_rate`**: 고객의 재구매 비율을 계산하여 `customers` 테이블에 업데이트하세요.
    - `repurchase_rate`는 각 고객이 2번 이상 주문한 제품 비율을 의미합니다. (즉, 재구매한 제품이 전체 구매 제품 중 몇 퍼센트를 차지하는지)

### 예시:

- 고객 A는 3번 주문을 했고, 그 중 2개의 제품을 재구매했습니다.
    - 평균 주문 금액: 100,000원
    - 총 지출 금액: 300,000원
    - 주문 횟수: 3번
    - 재구매 비율: 66.67%


**[정답 풀이]**
```
UPDATE customers c
SET
    avg_order_value = (
        SELECT AVG(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    total_spent = (
        SELECT SUM(od.quantity * od.unit_price)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    ),
    num_orders = (
        SELECT COUNT(DISTINCT o.order_id)
        FROM orders o
        WHERE o.customer_id = c.customer_id
    ),
    repurchase_rate = (
        SELECT
            COUNT(DISTINCT CASE WHEN od.product_id IN (
                SELECT product_id
                FROM order_details
                JOIN orders o ON order_details.order_id = o.order_id
                WHERE o.customer_id = c.customer_id
                GROUP BY order_details.product_id
                HAVING COUNT(order_details.product_id) > 1
            ) THEN od.product_id END) * 1.0 / COUNT(DISTINCT od.product_id)
        FROM order_details od
        JOIN orders o ON od.order_id = o.order_id
        WHERE o.customer_id = c.customer_id
    );
```

### 추가 질문:

1. 정답 쿼리에서 `repurchase_rate`를 구할 때 사용한 `HAVING COUNT(order_details.product_id) > 1`의 의미는 무엇인가요?
```
HAVING COUNT(order_details.product_id) > 1 조건은 주문한 제품 중에서 두 번 이상 구매한 제품만을 선택하는 역할을 한다. 이 조건은 고객이 재구매한 제품을 찾기 위해 사용된다.

구체적으로:

order_details.product_id는 고객이 주문한 제품을 나타내므로, 이 조건은 각 제품이 고객에 의해 두 번 이상 주문된 경우에만 해당 제품을 결과로 포함시키도록 한다. 즉, 재구매된 제품을 식별하는 데 사용된다.

```

2. 이 문제에서 사용될 수 있는 성능을 최적화하기 위한 방법은 무엇일까요?
```
이 쿼리는 서브쿼리와 조인을 여러 번 사용하는 방식이기 때문에 성능이 저하될 수 있다. 성능을 최적화하는 방법은 다음과 같다:

- 서브쿼리 최소화: 현재 쿼리는 고객별로 각 서브쿼리를 실행하고 있는데, 서브쿼리가 반복되므로 성능이 나빠질 수 있다. 대신 고객별로 중간 결과를 미리 계산하여 저장한 후, 이를 기반으로 업데이트를 수행하는 것이 더 효율적일 수 있다.
예를 들어, 고객별로 avg_order_value, total_spent, num_orders, repurchase_rate를 미리 계산하여 중간 테이블에 저장하고, 그 테이블을 기반으로 한 번에 업데이트하는 방법이다.

- 인덱스 추가: 주로 사용되는 컬럼(customer_id, product_id, order_id)에 인덱스를 추가하면 쿼리 성능을 향상시킬 수 있다.
예를 들어, orders.customer_id, order_details.product_id, order_details.order_id에 인덱스를 추가하여 데이터 검색을 더 빠르게 할 수 있다.

- JOIN 최적화: JOIN을 사용할 때 필요 없는 데이터까지 가져오지 않도록 최적화한다. 예를 들어, orders와 order_details에서 필요한 컬럼만 SELECT하도록 제한하거나, DISTINCT를 최소화하여 성능을 개선할 수 있다.

- 집계 함수 최적화: COUNT(DISTINCT ...), SUM(), AVG()와 같은 집계 함수는 대량의 데이터를 처리할 때 성능을 저하시킬 수 있다. 이러한 집계 함수가 불필요하게 반복되지 않도록 미리 계산해놓은 값을 사용하는 방법이 성능을 높이는 데 도움이 된다.

- 배치 처리: 만약 고객 수가 많다면, 한 번에 업데이트를 하지 않고 배치 처리로 나누어 실행하여 DB 서버에 과부하를 주지 않도록 할 수 있다.
```