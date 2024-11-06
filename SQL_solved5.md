# 틀린 코드 이유 분석
틀린 코드
```
SELECT *
FROM (SELECT FOOD_TYPE, REST_ID, REST_NAME, MAX(FAVORITES) AS FAVORITES
FROM REST_INFO
GROUP BY FOOD_TYPE
ORDER BY FOOD_TYPE DESC
```
틀린 이유: GROUP BY는 MAX함수까지 커버해줄 수 없기 때문이다. 최빈값이 아닌, 최상단의 값을 그냥 출력한다. 따라서 서브쿼리를 사용해서 해줘야 함.

정답 코드
```
SELECT FOOD_TYPE, REST_ID, REST_NAME, FAVORITES
FROM REST_INFO
WHERE (FOOD_TYPE, FAVORITES) IN (
    SELECT FOOD_TYPE, MAX(FAVORITES)    
    FROM REST_INFO
    GROUP BY FOOD_TYPE
) 
ORDER BY FOOD_TYPE DESC;
```


# 개선된 쿼리 학습


### 코드 해석

```sql
WITH RankedRest AS (  --1. RankedRest라는 임시 뷰(CTE)를 생성.
    SELECT 
        FOOD_TYPE,       
        REST_ID,    
        REST_NAME,      
        FAVORITES,         
        ROW_NUMBER() OVER (PARTITION BY FOOD_TYPE ORDER BY FAVORITES DESC, REST_ID) AS rnk  
        -- 2. 각 FOOD_TYPE 별로 FAVORITES를 내림차순으로 정렬한 후,
        --    같은 FAVORITES 값을 가진 경우 REST_ID로 추가 정렬.
        -- 3. 각 FOOD_TYPE 내에서 FAVORITES가 가장 높은 레코드에는 rnk = 1이 할당.
    FROM REST_INFO
)
SELECT 
    FOOD_TYPE,     
    REST_ID,       
    REST_NAME,       
    FAVORITES        
FROM RankedRest
WHERE rnk = 1       -- 4. 각 FOOD_TYPE에서 FAVORITES가 가장 높은 레스토랑만 선택.
ORDER BY FOOD_TYPE DESC;  -- 5. FOOD_TYPE 기준 내림차순으로 결과를 정렬.
```

### 이 코드의 장점


1. **명확한 그룹화**: `ROW_NUMBER()`는 각 `FOOD_TYPE` 별로 `FAVORITES`가 가장 높은 레스토랑을 효율적으로 찾을 수 있게 함. `ROW_NUMBER()`는 `PARTITION BY`를 사용하여 특정 기준으로 그룹을 나눈 후 각 그룹 내에서 순위를 지정하므로, 중첩 서브쿼리와 비교해 코드의 흐름이 명확해짐.

2. **성능 향상**: 중첩 서브쿼리를 사용하는 방식에서는 `MAX(FAVORITES)` 조건에 맞는 행을 찾기 위해 같은 테이블에 여러 번 접근해야 함. 반면, `ROW_NUMBER()` 방식에서는 테이블에 한 번만 접근하여 순위를 매기기 때문에 성능이 향상될 수 있음.

3. **확장성 및 유지 보수 용이성**: `WITH` 절(CTE)은 쿼리를 논리적 단계로 나누어 읽기 쉽게함. 이 방식은 쿼리의 일부를 변경해야 할 때 더 쉽게 수정할 수 있어 유지 보수가 편리함.

4. **다중 정렬 기준**: `ROW_NUMBER()`를 사용할 경우, `ORDER BY FAVORITES DESC, REST_ID`처럼 다중 정렬 기준을 설정할 수 있음. `MAX()` 함수와는 달리, 순위가 같은 경우 특정 열을 기준으로 추가 정렬을 할 수 있음.

따라서, `WITH` 절과 `ROW_NUMBER()`를 사용하는 방식은 중첩 쿼리보다 가독성, 성능, 확장성에서 모두 유리함.