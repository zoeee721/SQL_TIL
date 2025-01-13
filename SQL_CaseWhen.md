# 1. 온라인 쇼핑몰의 월 별 매출액 집계

UK E-Commerce Orders 데이터베이스는 영국의 한 온라인 쇼핑몰의 운영 데이터베이스 입니다. 그 중 orders 테이블은 온라인 쇼핑몰의 주문 정보를 담고 있고, order_items 테이블에는 주문한 상품, 상품의 개당 가격, 주문한 수량 등 주문의 상세 정보가 저장되어 있습니다.

이 온라인 쇼핑몰의 월 별 매출 규모를 한 눈에 파악할 수 있는 데이터를 만들고 싶습니다. 위 두 테이블의 데이터를 조합해 월 별로 취소 주문을 제외한 주문 금액의 합계, 취소 주문의 금액 합계, 그리고 총 합계를 계산하는 쿼리를 작성해주세요. order_id가 C로 시작하는 주문이 취소 주문입니다. 결과 데이터는 아래 4개 컬럼을 포함해야 하고 order_month 컬럼의 값으로 오름차순 정렬되어 있어야 합니다.



#### [ 결과 데이터 컬럼 ]
- order_month: YYYY-MM 형식으로 표기된 주문 연, 월 정보
- ordered_amount: 취소 주문을 제외한 주문 금액의 합계
- canceled_amount: 취소 주문의 금액 합계 (음수로 표시)
- total_amount: 취소 주문을 포함한 주문 금액의 총 합계

### [ 문제 풀이를 위해 생각해볼 것들 ]
1. date 컬럼에서 년, 월 정보만 가져오는 방법.
   ```
   DATE_FORMAT 함수 이용하기?
   ex. DATE_FORMAT(O.order_date, "%Y-%m")

   하지만, SQLite에서는 DATE_FORMAT 함수가 없고,
   strftime 함수를 이용해주어야 함.
   ex. strftime('%Y-%m', O.order_date) as order_month
   ```
2. C로 시작하는 환불한 거래 제외하고 금액 카운트하는 조건 설정하기.
   ```
   WHERE 절에 
   O.order_id NOT LIKE "C%"
   조건 포함 해주기.
   -> 이 방법을 처음에 시도했는데.. 굉장히 복잡해져서 그냥

   select문에 case when 써주는 걸로..
   ex. SELECT sum(CASE WHEN OI.order_id NOT LIKE 'C%' THEN OI.price*OI.quantity ELSE 0 END) as ordered_amount

   ```

### 최종 정답 코드
```SQL
SELECT strftime('%Y-%m', O.order_date) as order_month,
  sum(CASE WHEN OI.order_id NOT LIKE 'C%' THEN OI.price*OI.quantity ELSE 0 END) as ordered_amount,
  sum(CASE WHEN OI.order_id LIKE 'C%' THEN OI.price*OI.quantity ELSE 0 END) as canceled_amount,
  sum(OI.price*OI.quantity) as total_amount
FROM orders as O
JOIN order_items as OI
ON O.order_id = OI.order_id
GROUP BY 1
ORDER BY 1
```
![img](/image_w8/1answer.png)

### 함수 및 문법 정리

#### DATE_FORMAT(변수, '형식') ↔ strftime('형식', 변수)
```
ex. DATE_FORMAT(O.order_date, "%Y-%m")
ex. strftime('%Y-%m', O.order_date)

SQLite에서는 strftime 함수만 사용 가능.
```
#### CASE WHEN 함수
```SQL
[기본 문법]

CASE
    WHEN 조건1 THEN 반환값1
    WHEN 조건2 THEN 반환값2
    ...
    ELSE 반환값N
END

--------------------------------------------------------

[ex. 집계함수와 함께 사용]

SELECT 
    SUM(CASE WHEN status = 'Completed' THEN price ELSE 0 END) AS completed_sales,
    SUM(CASE WHEN status = 'Canceled' THEN price ELSE 0 END) AS canceled_sales
FROM orders;

--------------------------------------------------------

[ex. 컬럼값 변경]

SELECT 
    customer_id,
    CASE 
        WHEN country = 'US' THEN 'United States'
        WHEN country = 'KR' THEN 'South Korea'
        ELSE 'Other'
    END AS country_name
FROM customers;

--------------------------------------------------------

[유의할 점]

조건이 참인 첫 번째 WHEN만 실행됨.
즉, 여러 조건이 참이어도 첫 번째 조건만 적용.

ex. 
SELECT 
    customer_id,
    total_spent,
    CASE 
        WHEN total_spent >= 1000 THEN 'VIP'
        WHEN total_spent >= 500 THEN 'Gold'
        WHEN total_spent >= 100 THEN 'Silver'
        ELSE 'Bronze'
    END AS customer_tier
FROM customers;

```

# 2. 가구 판매의 비중이 높았던 날 찾기

US E-Commerce Records 2020 데이터셋은 미국 이커머스 웹사이트의 판매 데이터 입니다. 이 중 records 테이블은 주문 번호, 주문 날짜, 주문 지역, 카테고리 등 주문의 상세 정보를 담고 있습니다. 이 데이터를 이용하여 가구 판매의 비중이 높았던 날을 찾고 싶습니다. 일별 주문 수가 10개 이상인 날 중에서, ‘Furniture’ 카테고리 주문의 비율이 40% 이상 이었던 날만 출력하는 쿼리를 작성해주세요. 카테고리 정보는 category 컬럼에 기록되어 있습니다.

결과 데이터는 아래의 컬럼들을 포함해야 합니다. Furniture 카테고리의 주문 비율은 백분율로 계산하며, 반올림하여 소수점 둘째자리까지만 출력해주세요. Furniture 카테고리의 주문 비율이 높은 것부터 보여주도록 정렬하고, 비율이 같다면 날짜 순으로 정렬해주세요.


#### [ 결과 데이터 컬럼 ]
- order_date: 주문 날짜
- furniture: 해당 일의 Furniture 카테고리 주문 수
- furniture_pct: 해당 일의 전체 주문 대비 Furniture 카테고리 주문의 비율 (%)


### [ 문제 풀이를 위해 생각해볼 것들 ]
1. 일별 주문 수가 10개 이상인 날 중에서, ‘Furniture’ 카테고리 주문의 비율이 40% 이상 이었던 날만 출력하는 조건
   ```

   ```


2. 뭘까용용
   ```

   ```

### 최종 정답 코드
```SQL

```
![img]

### 함수 및 문법 정리