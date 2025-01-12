# 온라인 쇼핑몰의 월 별 매출액 집계

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
2. C로 시작하는 환불한 거래 ID 제외하고 금액 카운트하는 조건 설정하기.
   ```
   WHERE 절에 
   O.order_id IS NOT "C%"
   조건 포함 해주기.

   select문에
   COUNT(O.order_id) as ordered_amount
   ```


### 함수 및 문법 정리

#### DATE_FORMAT(변수, '형식') ↔ strftime('형식', 변수)
```
ex. DATE_FORMAT(O.order_date, "%Y-%m")
ex. strftime('%Y-%m', O.order_date)

SQLite에서는 strftime 함수만 사용 가능.
```