# SQL_ADVANCED 3주차 정규 과제 

## Week 3: 윈도우 함수 (Window Functions)

📌**SQL_ADVANCED 정규과제**는 매주 정해진 주제에 따라 **MySQL 공식 문서 또는 한글 블로그 자료를 참고해 개념을 정리한 후, 이번 주차에는 LeetCode SQL 문제 3문제**와 **추가 확인문제**를 직접 풀어보며 학습하는 과제입니다. 

이번 주는 아래의 **SQL_ADVANCED_3rd_TIL**에 나열된 주제를 중심으로 개념을 학습하고, 주차별 **학습 목표**에 맞게 정리해주세요. 정리한 내용은 GitHub에 업로드한 후, **스프레드시트의 'SQL' 시트에 링크를 제출**해주세요. 



**(수행 인증샷은 필수입니다.)** 

> Leet code의 문제를 풀고 '정답입니다' 문구를 캡쳐해서 올려주시면 됩니다. 



## SQL_ADVANCED_3rd

### 14.20.2 Window Function Concepts and Syntax

### 14.20.1 Window Function Description

### 14.19.1 Aggregate Function Descriptions

- 위 문서 중 *14.19.1. Aggregate Function Descriptions* 문서에서 1주차 (집계 함수) 에서 다룬 부분을 제외하고 **OVER( ) 절을 활용한 윈도우 함수 문법과 `RANK( ), DENSE_RANK( ), ROW_NUMBER( ), LAG( ), LEAD( )`등 윈도우 함수 특유의 기능 중심으로 정리해주세요.**



## 🏁 강의 수강 (Study Schedule)

| 주차  | 공부 범위               | 완료 여부 |
| ----- | ----------------------- | --------- |
| 1주차 | 서브쿼리 & CTE          | ✅         |
| 2주차 | 집합 연산자 & 그룹 함수 | ✅         |
| 3주차 | 윈도우 함수             | ✅         |
| 4주차 | Top N 쿼리              | 🍽️         |
| 5주차 | 계층형 질의와 셀프 조인 | 🍽️         |
| 6주차 | PIVOT / UNPIVOT         | 🍽️         |
| 7주차 | 정규 표현식             | 🍽️         |



### 공식 문서 활용 팁

>  **MySQL 공식 문서는 영어로 제공되지만, 크롬 브라우저에서 공식 문서를 열고 이 페이지 번역하기에서 한국어를 선택하면 번역된 버전으로 확인할 수 있습니다. 다만, 번역본은 문맥이 어색한 부분이 종종 있으니 영어 원문과 한국어 번역본을 왔다 갔다 하며 확인하거나, 교육팀장의 정리 예시를 참고하셔도 괜찮습니다.**



# 1️⃣ 학습 내용

> 아래의 링크를 통해 *MySQL 공식문서*로 이동하실 수 있습니다.
>
> - 14.20.2 Window Function Concepts and Syntax : MySQL 공식문서
>
> https://dev.mysql.com/doc/refman/8.0/en/window-functions-usage.html
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-Window-Function/
>
> - 14.20.1 Window Function Description : MySQL 공식문서
>
> https://dev.mysql.com/doc/refman/8.0/en/window-function-descriptions.html
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-Window-Function(2)/
>
> - 14.19.1 Aggregate Function Descriptions : MySQL 공식문서
>
> https://dev.mysql.com/doc/refman/8.0/en/aggregate-functions.html
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-aggregate_function/
>

<br>



---

# 2️⃣ 학습 내용 정리하기

# 1. 윈도우 함수

~~~
✅ 학습 목표 :
* OVER 절을 통해 행 단위 분석을 가능하게 하는 윈도우 함수의 구조를 이해한다.
* RANK, DENSE_RANK, ROW_NUMBER의 차이를 구분하고 사용할 수 있다.
* 이전 또는 이후 행을 참조하는 LAG, LEAD 함수를 적절히 사용할 수 있다.
~~~

## 윈도우 함수 기본 개념
- 집계 함수: 여러 행을 하나의 결과 행으로 축소 (ex. SUM(profit) -> **전체 합계**)
- 윈도우 함수: 각 행마다 결과를 반환 (**행 단위**)
- 현재 행: 함수 계산이 수행되는 대상 행
- 윈도우(window): 현재 행을 기준으로 관련된 행들의 집합

기존 계산
```
SELECT SUM(profit) AS total_profit FROM sales;
-- 전체 합계 = 7535

SELECT country, SUM(profit) AS country_profit
FROM sales
GROUP BY country
ORDER BY country;
-- 국가별 합계: Finland=1610, India=1350, USA=4575
```

윈도우 함수를 사용할 때 
```sql
SELECT
  year, country, product, profit,
  SUM(profit) OVER() AS total_profit, -- over(): 전채 행을 하나의 파티션으로 취급, 각 행에 전체 합계 표시
  SUM(profit) OVER(PARTITION BY country) AS country_profit
FROM sales -- country 별 partition 합계 표시
ORDER BY country, year, product, profit;
```
- GROUP BY: 행이 줄어듦(요약된 결과)
- OVER(): 원래 행을 유지하면서, 추가적으로 집계값을 계산

## 사용 위치와 실행 순서
- 윈도우 함수는 SELECT 목록과 ORDER BY 절에서만 사용 가능
- 실행 순서: FROM -> WHERE -> GROUP BY -> HAVING -> WINDOW FTN -> ORDER BY -> LIMIT

## 윈도우 함수로 사용 가능한 집계 함수
- OVER 절이 있으면 윈도우 함수, 없으면 일반 집계 함수로 동작

| 함수 | 설명 |
|------|------|
| AVG() | 평균 |
| BIT_AND(), BIT_OR(), BIT_XOR() | 비트 연산 집계 |
| COUNT() | 개수 |
| JSON_ARRAYAGG(), JSON_OBJECTAGG() | JSON 배열/객체 집계 |
| MAX(), MIN() | 최댓값 / 최솟값 |
| STDDEV_POP(), STDDEV(), STD() | 모표준편차 |
| STDDEV_SAMP() | 표본표준편차 |
| SUM() | 합계 |
| VAR_POP(), VARIANCE() | 모분산 |
| VAR_SAMP() | 표본분산 |

## 비집계 전용 윈도우 함수
- 윈도우 함수로만 사용 가능하며 OVER() 절이 필수

| 함수 | 설명 |
|------|------|
| CUME_DIST() | 누적 분포 (누적 백분위) |
| DENSE_RANK() | 순위(동점은 같은 순위, 다음은 연속) |
| FIRST_VALUE() | 파티션 내 첫 번째 값 |
| LAG(), LEAD() | 이전 행 값 / 다음 행 값 |
| LAST_VALUE() | 파티션 내 마지막 값 |
| NTH_VALUE() | 파티션 내 N번째 값 |
| NTILE() | 파티션을 N개 구간으로 분할 |
| PERCENT_RANK() | 백분위 순위 |
| RANK() | 순위(동점 시 건너뜀) |
| ROW_NUMBER() | 순차 행 번호 |


```sql
SELECT
  year, country, product, profit,
  ROW_NUMBER() OVER(PARTITION BY country) AS row_num1,
  ROW_NUMBER() OVER(PARTITION BY country ORDER BY year, product) AS row_num2
FROM sales;
``` 
- row_num1: 정렬 없는 파티션 -> 비결정적 번호 부여 
- row_num2: 정렬 기준 추가 -> 일관된 순서 부여 

## OVER 절 문법
```sql
over_clause:
    {OVER (window_spec) | OVER window_name}
```
- OVER(window_spec): 괄호 안에 직접 윈도 정의
- OVER window_name: WINDOW 절에서 정의한 이름 참조

## window_spce 구성 요소
```sql
window_spec:
    [window_name] [partition_clause] [order_clause] [frame_clause]
```
- window_name: WINDOW 절에서 정의된 윈도 이름
- partition_clause: PARTITION BY로 행을 그룹화 (표준 SQL은 컬럼만 허용, MySQL은 표현식도 허용)
- order_clause: 각 파티션 내 행의 순서 지정
- frame_clause: 파티션 내 “프레임(부분집합)” 정의 (→ 14.20.3 Frame Specification)

## 윈도우 함수 목록

| 이름            | 설명                                    |
|-----------------|---------------------------------------|
| CUME_DIST()     | 누적분포값 (현재 값 이하 비율)                |
| DENSE_RANK()    | 파티션 내 현재 행의 순위 (동점 간격 없음)        |
| FIRST_VALUE()   | 프레임 첫 행의 값                          |
| LAG()           | 파티션에서 현재 행보다 N행 이전의 값           |
| LAST_VALUE()    | 프레임 마지막 행의 값                       |
| LEAD()          | 파티션에서 현재 행보다 N행 이후의 값           |
| NTH_VALUE()     | 프레임 N번째 행의 값                        |
| NTILE()         | 파티션을 N개 버킷으로 나눈 버킷 번호            |
| PERCENT_RANK()  | 백분위 순위 (0~1)                        |
| RANK()          | 파티션 내 현재 행의 순위 (동점 간격 발생)        |
| ROW_NUMBER()    | 파티션 내 현재 행의 일련번호                  |

- over_clause는 OVER(...) 구문을 의미
- 일부 함수는 null_treatment 옵션이 있음

## 함수별 설명
`CUME_DIST() over_clause`
- 파티션 내 현재 행 값 이하의 비율 반환, 범위: 0~1
- ORDER BY 권장: 없으면 모든 행이 피어로 간주되어 값은 N/N = 1 (N = 파티션 크기)
```sql
SELECT 
  val,
  ROW_NUMBER()   OVER w AS row_number, -- 고유한 일련번호를 매김
  CUME_DIST()    OVER w AS cume_dist, -- CUME_DIST(): 누적 분포 값
  PERCENT_RANK() OVER w AS percent_rank -- PERCENT_RANK(): 현재 행의 백분위순위를 계산
FROM numbers
WINDOW w AS (ORDER BY val);  -- ORDER BY val 기준으로 행을 정렬한 뒤 
``` 

## DENSE_RANK() over_clause

- 파티션 내 순위를 반환하되 동점 간격이 생기지 않도록 연속 순위 부여
- ORDER BY 권장(없으면 모두 피어)

## FIRST_VALUE(expr)[null_treatment] over_clause
- 프레임 첫 행의 expr 값을 반환
- 프레임 정의에 따라 결과가 달라짐
```sql
SELECT
  time, subject, val,
  FIRST_VALUE(val)  OVER w AS first,  -- FIRST_VALUE(): 가장 처음인 행의 값 반환
  LAST_VALUE(val)   OVER w AS last,  -- LAST_VALUE(): 마지막 행의 값 반환
  NTH_VALUE(val, 2) OVER w AS second,  -- NTH_VALUE(col, n): n번째 값 반환
  NTH_VALUE(val, 4) OVER w AS fourth
FROM observations
WINDOW w AS (
  PARTITION BY subject ORDER BY time
  ROWS UNBOUNDED PRECEDING
);
```

## LAG(expr [, N[, default]]) [null_treatment] over_clause

- 파티션에서 현재 행 기준 N행 이전의 expr 값을 반환
- 해당 행이 없으면 default 반환, 기본값: N=1, default=NULL

1. 차분/차이 계산
```sql
SELECT
  t, val,
  LAG(val)        OVER w AS lag,
  LEAD(val)       OVER w AS lead,
  val - LAG(val)  OVER w AS lag_diff,
  val - LEAD(val) OVER w AS lead_diff
FROM series
WINDOW w AS (ORDER BY t);
```

2. 피보나치 누적 아이디어
```sql 
SELECT
  n,
  LAG(n, 1, 0)      OVER w AS lag,
  LEAD(n, 1, 0)     OVER w AS lead,
  n + LAG(n, 1, 0)  OVER w AS next_n,
  n + LEAD(n, 1, 0) OVER w AS next_next_n
FROM fib
WINDOW w AS (ORDER BY n);
``` 

## LAST_VALUE(expr)[null_treatment] over_clause
- 프레임 마지막 행의 expr 값을 반환
- 프레임/정렬 정의에 따라 값이 달라짐

## LEAD(expr [, N[, default]]) [null_treatment] over_clause
- 파티션에서 현재 행 기준 N행 이후의 expr 값을 반환
- 해당 행이 없으면 default 반환. 기본값: N=1, default=NULL

## NTH_VALUE(expr, N) [from_first_last] [null_treatment] over_clause
- 프레임 N번째 행의 expr 값을 반환. 없으면 NULL
- N은 양의 정수 리터럴이어야 함

## NTILE(N) over_clause
- 파티션을 N개 버킷으로 분할 후, 현재 행의 버킷 번호(1~N) 반환
- N은 양의 정수 리터럴
- ORDER BY 권장: 없으면 피어 처리로 분할이 비결정적
```sql
SELECT
  val,
  ROW_NUMBER() OVER w AS row_number,
  NTILE(2)     OVER w AS ntile2,
  NTILE(4)     OVER w AS ntile4
FROM numbers
WINDOW w AS (ORDER BY val);
``` 

## PERCENT_RANK() over_clause
- 파티션 내 최대값을 제외한 값 기준 상대 백분위 순위 반환
- 공식: (rank - 1) / (rows - 1) (0~1).
- ORDER BY 권장

## RANK() over_clause
- 파티션 내 순위를 반환하되, 동점 시 같은 순위를 부여하며 간격(gaps) 발생
- ORDER BY 권장
RANK() vs DENSE_RANK()
```sql
SELECT
  val,
  ROW_NUMBER() OVER w AS row_number,
  RANK()       OVER w AS rank,
  DENSE_RANK() OVER w AS dense_rank
FROM numbers
WINDOW w AS (ORDER BY val);
```
- RANK(): 동점 다음 값의 순위가 동점 개수만큼 건너뜀
- DENSE_RANK(): 다음 값의 순위는 +1(간격 없음)

## ROW_NUMBER() over_clause
- 파티션 내 현재 행의 일련번호(1 ~ 파티션 행수) 반환
- ORDER BY가 번호 부여 순서를 결정, 없으면 비결정적
- 동점에 서로 다른 번호를 부여, 동점에 동일 값을 원하면 RANK()/DENSE_RANK() 사용.



---

# 3️⃣ 실습 문제

## LeetCode 문제 

https://leetcode.com/problems/department-top-three-salaries/

> LeetCode 185. Department Top Three Salaries 
>
> 학습 포인트 : DENSE_RANK( ) + PARTITION BY 사용으로 그룹 내 상위 N개 추출

https://leetcode.com/problems/consecutive-numbers/

> LeetCode 180. Consecutive Numbers 
>
> 학습 포인트 : LAG( ) 함수로 이전 값과 비교하여 연속 데이터 탐지 

https://leetcode.com/problems/last-person-to-fit-in-the-bus/

> LeetCode 2481. Last Person to Fit in the Bus 
>
> 학습 포인트 : SUM( ) OVER (ORDER BY ...) 로 누적 합계 계산 후 조건 필터링 



문제를 푸는 다양한 방법이 존재하지만, **윈도우 함수를 사용하여 해결하는 방식에 대해 고민해주시길 바랍니다.** 

---

## 문제 인증란

<!-- 이 주석을 지우고 여기에 문제 푼 인증사진을 올려주세요. -->



---

# 확인문제

## 문제 1

> **🧚예린이는 고객별로 얼마나 많은 주문을 하는지 분석하기 위해, 고객의 주문 목록에 주문 순서를 표시하는 쿼리를 작성해보았습니다. 이때 주문일 순서대로 각 고객의 주문 번호를 매기기 위해 윈도우 함수를 활용했습니다.**

~~~sql
SELECT customer_id, order_id, order_date,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_rank
FROM Orders;
~~~

> **이번에는 예린이에게 "윈도우 함수를 쓰지 않고 동일한 결과를 만들어보라"는 미션을 받았습니다. 예린이는 이 작업을 어떻게 해야할지 막막합니다. 예린이를 도와 ROW_NUMBER() 윈도우 함수 없이 동일한 결과를 서브쿼리나 JOIN을 사용해서 작성해보세요.**

~~~
여기에 답을 작성해주세요!
~~~



<br>

### 🎉 수고하셨습니다.