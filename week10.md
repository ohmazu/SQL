# SQL_ADVANCED 4주차 정규 과제 

## Week 4 : TOP-N 쿼리

📌**SQL_ADVANCED 정규과제**는 매주 정해진 주제에 따라 **MySQL 공식 문서 또는 한글 블로그 자료를 참고해 개념을 정리한 후, 프로그래머스, SolveSql, LeetCode 중에서 SQL 문제 4문제**와 **추가 확인문제**를 직접 풀어보며 학습하는 과제입니다. 

이번 주는 아래의 **SQL_ADVANCED_4th_TIL**에 나열된 주제를 중심으로 개념을 학습하고, 주차별 **학습 목표**에 맞게 정리해주세요. 정리한 내용은 GitHub에 업로드한 후, **스프레드시트의 'SQL' 시트에 링크를 제출**해주세요. 



**(수행 인증샷은 필수입니다.)** 

> 프로그래머스 문제를 풀고 '정답입니다' 문구를 캡쳐해서 올려주시면 됩니다. 



## SQL_ADVANCED_4th

### 15.2.13. SELECT Statement

- `ORDER BY, LIMIT, LIMIT and Subqueries` 중심으로 학습해주세요. 



## 🏁 강의 수강 (Study Schedule)

| 주차  | 공부 범위               | 완료 여부 |
| ----- | ----------------------- | --------- |
| 1주차 | 서브쿼리 & CTE          | ✅         |
| 2주차 | 집합 연산자 & 그룹 함수 | ✅         |
| 3주차 | 윈도우 함수             | ✅         |
| 4주차 | Top N 쿼리              | ✅         |
| 5주차 | 계층형 질의와 셀프 조인 | 🍽️         |
| 6주차 | PIVOT / UNPIVOT         | 🍽️         |
| 7주차 | 정규 표현식             | 🍽️         |



### 공식 문서 활용 팁

>  **MySQL 공식 문서는 영어로 제공되지만, 크롬 브라우저에서 공식 문서를 열고 이 페이지 번역하기에서 한국어를 선택하면 번역된 버전으로 확인할 수 있습니다. 다만, 번역본은 문맥이 어색한 부분이 종종 있으니 영어 원문과 한국어 번역본을 왔다 갔다 하며 확인하거나, 교육팀장의 정리 예시를 참고하셔도 괜찮습니다.**



# 1️⃣ 학습 내용

> 아래의 링크를 통해 *MySQL 공식문서*로 이동하실 수 있습니다.
>
> - 15.2.13 SELECT Statement : MySQL 공식문서 
>
> https://dev.mysql.com/doc/refman/8.0/en/select.html#order-by-optimization
>
> (한국어 버전) https://dart-b-official.github.io/posts/mysql-TopN/
>



<br>

<!-- 여기까진 그대로 둬 주세요-->

---

# 2️⃣ 학습 내용 정리하기

## 1. TOP N 쿼리

~~~
✅ 학습 목표 :
* LIMIT 와 ORDER BY를 이용한 TOP-N 쿼리 작성이 가능하다.
* SubQuery나 RANK 대신 LIMIT으로 간단한 순위 집계가 가능함을 이해한다. 
~~~

## SELECT
`SELECT`문
- 한 개 이상의 테이블에서 행 조회
- UNION 연산자, 서브쿼리, 공통 테이블 표현식(CTE, WITH 절)과 함께 사용 가능
```sql
SELECT [option] col
FROM table
[WHERE condition]
[GROUP BY col [WITH ROLLUP]]
[HAVING group condition] -- 그룹핑된 결과에 조건 걸기(집계함수와 함께 사용)
[WINDOW name AS spec]
[ORDER BY col [ASC|DESC]]
[LIMIT row_count] -- 몇 개만 가져올지 제한
[FOR UPDATE / SHARE]
```
### SELECT 절 주요 옵션

| 옵션                     | 설명                                                                 |
|--------------------------|----------------------------------------------------------------------|
| ALL (기본값)             | 모든 행을 반환, 중복 포함                                             |
| DISTINCT / DISTINCTROW   | 중복 제거 후 행 반환                                                  |
| HIGH_PRIORITY            | SELECT 실행을 UPDATE보다 우선 처리 (MyISAM 등 테이블 수준 잠금 엔진만 해당) |
| STRAIGHT_JOIN            | 옵티마이저가 아닌 작성 순서대로 조인 실행                              |
| SQL_SMALL_RESULT         | 결과가 작음을 힌트 → 메모리 임시 테이블 사용                          |
| SQL_BIG_RESULT           | 결과가 큼을 힌트 → 디스크 기반 임시 테이블/정렬 사용                  |
| SQL_BUFFER_RESULT        | 결과를 임시 테이블에 버퍼링 → 테이블 잠금 빨리 해제                   |
| SQL_CALC_FOUND_ROWS      | LIMIT 무시한 전체 행 수 계산 (→ FOUND_ROWS()로 조회 가능, 8.0.17부터 deprecated) |
| SQL_NO_CACHE             | 쿼리 캐시 비활성화 (8.0 이후 효과 없음, deprecated)                   |

- PARTITION: 하나의 테이블을 여러 개 파티션으로 분할 저장, 조회는 하나의 테이블처럼 가능하고 내부적으로는 행이 조건에 따라 각 파티션에 저장
- WHERE: 행을 필터링하는 조건, **집계 함수 사용 불가**
- INTO: 결과를 파일/변수에 저장
    - INTO OUTFILE, INTO DUMPFILE, INTO var_name ...
- FOR UPDATE/FOR SHARE: 조회된 행을 잠금, 트랜잭션 종료 시까지 유지

```sql
-- 단순 계산
SELECT 1 + 1;
-- 2

-- DUAL 테이블 (더미 테이블) 사용
SELECT 1 + 1 FROM DUAL;
-- 2

-- 컬럼 alias
SELECT CONCAT(last_name, ', ', first_name) AS full_name
FROM mytable ORDER BY full_name;

-- GROUP BY + HAVING
SELECT user, MAX(salary) 
FROM users
GROUP BY user
HAVING MAX(salary) > 10;

-- LIMIT 사용
SELECT * FROM tbl LIMIT 5;        -- 처음 5행
SELECT * FROM tbl LIMIT 5, 10;    -- 6행부터 10개
```

<br>
<br>

---

# 3️⃣ 실습 문제

## 문제 

https://leetcode.com/problems/rank-scores/

> LeetCode 178. Rank Scores
>
> 학습 포인트 : DENSE_RANK( )를 활용하여 점수별 순위 부여, 동점자 처리, 윈도우 함수 복습 
```sql
SELECT 
    s.score,
    (SELECT COUNT(DISTINCT score) 
     FROM Scores 
     WHERE score >= s.score) AS `rank`
FROM Scores s
ORDER BY s.score DESC;
```
- `WHERE score >= s.score`: 현재 행의 점수보다 크거나 같은 점수가 몇 개의 다른 값이 있는지 계산
- `COUNT(DISTINCT score)`: 서로 다른 점수 개수만 세기 때문에 동점은 같은 등수를 가짐
- `AS rank`: 위에서 계산한 개수를 등수로 표시 (예약어라 백틱으로 감쌈)


https://school.programmers.co.kr/learn/courses/30/lessons/133027

> 프로그래머스 : 주문량이 많은 아이스크림들 조회하기 (Lev 4)
>
> Hint
>
> - 문제 핵심은 '총 주문량 합산' 입니다. 
>
> - 두 테이블을 '세로로' 합쳐야합니다. 
>   - 저희는 이 부분을 1주차에 `UNION ALL` 을 통해 방법을 배웠습니다. 
> - 합쳐진 테이블에서 FLAVOR 별로 그룹화해 주문량을 합산하세요. 
> - 상위 3개를 추출 = 주문량 기준으로 내림차순하여 이번에 학습한 것을 사용해야 합니다. 
```sql
SELECT f.FLAVOR
FROM FIRST_HALF AS f
LEFT JOIN (
  SELECT FLAVOR, SUM(TOTAL_ORDER) AS july_sum
  FROM JULY
  GROUP BY FLAVOR
) AS j
  ON j.FLAVOR = f.FLAVOR
ORDER BY (f.TOTAL_ORDER + IFNULL(j.july_sum, 0)) DESC,
         f.FLAVOR ASC   
LIMIT 3;
``` 
1. 서브쿼리
- `SELECT FLAVOR, SUM(TOTAL_ORDER) AS july_sum`
- 7월 한 달 동안 맛별 총 주문량 생성
2. LEFT JOIN
- `FIRST_HALF`와 `J`를 `FALVOR` 기준으로 합침
- 7월에 주문이 없는 맛도 포함시키기 위해 LEFT JOIN 사용
3. 합계 계산
- `f.TOTAL_ORDER + IFNULL(j.july_sum, 0)`
- 상반기 주문량 + 7월 주문량 = 전체 주문량
4. 정렬&상위 3개 추출
- 합계 기준 내림차순 정렬
- 같은 합계일 때는 FLAVOR 이름순으로 정렬
- `LIMIT 3`: 상위 3개 맛만 출력

---

## 문제 인증란

![1001](sql_images/1001.png)
![1002](sql_images/1002.png)


---

# 확인문제

## 문제 1

> **🧚영리는 지역별로 가장 인기 있는 식당 2곳씩을 뽑기 위해 다음과 같은 UNION ALL 기반 쿼리를 작성했습니다.**

~~~sql
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '서울'
  ORDER BY review_count DESC
  LIMIT 2
)
UNION ALL
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '부산'
  ORDER BY review_count DESC
  LIMIT 2
)
UNION ALL
(
  SELECT region, restaurant_name, review_count
  FROM Restaurants
  WHERE region = '대구'
  ORDER BY review_count DESC
  LIMIT 2
);
~~~

> **쿼리는 잘 작동하긴 하지만, 지역을 더 추가해달라는 재원이의 부탁으로 UNION ALL 블록을 계속 추가하게 되어 관리가 어려울 것 같아서 힘들어하고 있었습니다. 여러분들은 이 쿼리를 윈도우 함수로 변경하여 더 쉽게 리팩토링을 하려고 합니다. 민서를 도와서 UNION ALL 없이 RANK( ) 또는 ROW_NUMBER( ) 윈도우 함수를 사용해, 각 지역별로 리뷰 수가 가장 많은 상위 2개 식당을 추출하는 쿼리를 작성해보세요.**




```sql
SELECT region, restaurant_name, review_count
FROM (
    SELECT 
        region,
        restaurant_name,
        review_count,
        ROW_NUMBER() OVER (  -- 지역별로 리뷰 수 내림차순 정렬
            PARTITION BY region 
            ORDER BY review_count DESC
        ) AS rn  -- 각 지역 안에서 순위 번호(rn)를 붙임
    FROM Restaurants
) t
WHERE rn <= 2;  -- 지역별 상위 2개만 선택
```



<br>

### 🎉 수고하셨습니다.