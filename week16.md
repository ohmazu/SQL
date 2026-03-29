# SQL_MASTER 4주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_4th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_4th_TIL

### 5장 사용자를 파악하기 위한 데이터 추출
#### 1. 사용자 전체의 특징과 경향 찾기


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.20~50    | ✅         |
| 2주차 | p.52~136   | ✅         |
| 3주차 | p.138~184  | ✅         |
| 4주차 | p.186~232 | ✅         |
| 5주차 | p.233~321 | 🍽️         |
| 6주차 | p.324~406 | 🍽️         |
| 7주차 | p.408~464 | 🍽️         |

<br>

<!-- 여기까진 그대로 둬 주세요-->


# 실습

## 0. 실습 규칙

1. 샘플 데이터 생성 코드는 **07_SQL_MASTER_Template/src** 경로에 장별로 정리되어 있습니다.
2. 아래 목차에 맞춰 해당 코드를 실행하여 샘플 데이터를 생성한 후, 각 장에서 요구하는 쿼리를 직접 작성해보시기 바랍니다.
3. 작성한 쿼리의 **실행 결과 화면도 함께 제출**해 주세요.
4. 단순히 교재의 예시 코드를 그대로 작성하는 것이 아니라, **제시된 로직을 충분히 이해한 뒤 교재를 보지 않고 스스로 쿼리를 구성**해보는 것을 권장합니다.
5. 교재 예시는 PostgreSQL, Hive, BigQuery 등 다양한 DBMS 기준으로 제시되어 있기 때문에, **MySQL이 아닌 다른 SQL 환경을 사용하여 실습을 진행해도 무방합니다.**
6. 다만, 사용 중인 DBMS에 맞는 문법으로 적절히 변환하여 작성하시기 바랍니다.

## 1. 사용자 전체의 특징과 경향 찾기

### 1-1 사용자의 액션 수 집계하기

#### 사용자 액션 집계(핵심 KPI 만들기)
- 목적
    - 유저 행동 파악(view -> add_cart -> purchase 퍼널)


```sql
WITH stats AS (
    -- 전체 유저 수
    SELECT COUNT(DISTINCT session) AS total_uu
    FROM action_log
)

SELECT
    l.action,
    COUNT(DISTINCT l.session) AS action_uu,
    COUNT(*) AS action_count,
    s.total_uu,

    -- 사용률 (%)
    100.0 * COUNT(DISTINCT l.session) / s.total_uu AS usage_rate,

    -- 유저당 평균 액션 수
    1.0 * COUNT(*) / COUNT(DISTINCT l.session) AS count_per_user

FROM action_log l
CROSS JOIN stats s
GROUP BY l.action, s.total_uu
ORDER BY action_count DESC;
```
- CROSS JOIN: 전체 유저 수를 모든 row에 붙이기
- 퍼널 분석 기본 구조

#### 로그인 상태별 액션 집계 + 전체 포함(ROLLUP)

- 목적
    - 회원 vs 비회원 행동 차이 분석
```sql
WITH action_log_with_status AS (
    SELECT
        session,
        user_id,
        action,

        CASE 
            WHEN COALESCE(user_id, '') <> '' THEN 'login'
            ELSE 'guest'
        END AS login_status

    FROM action_log
)

SELECT *
FROM action_log_with_status;
``` 

### 1-2 연령별 구분 집계하기
#### 연령 계산
- 왜 중요한지?
    - 나이 컬럼은 시간 지나면 틀어짐 -> 생일 기준으로 계산 필요
- 방법
    - 날짜를 정수로 변환 후 차이 계산

```sql
WITH mst_users_with_int_birth_date AS (
    SELECT
        *,
        20170101 AS int_specific_date,

        -- 생일 문자열 → 정수 변환
        CAST(REPLACE(SUBSTRING(birth_date, 1, 10), '-', '') AS INT) AS int_birth_date

    FROM mst_users
),

mst_users_with_age AS (
    SELECT
        *,
        FLOOR((int_specific_date - int_birth_date) / 10000) AS age
    FROM mst_users_with_int_birth_date
)

SELECT
    user_id,
    sex,
    birth_date,
    age
FROM mst_users_with_age;
```

#### 연령 + 성별 카테고리 만들기
- 목적
    - CRM / 마케팅 핵심 세그먼트 생성

```sql
WITH mst_users_with_age AS (
    -- 위 age 계산 코드
    SELECT * FROM ...
),

mst_users_with_category AS (
    SELECT
        user_id,
        sex,
        age,

        CONCAT(
            CASE 
                WHEN age >= 20 THEN sex
                ELSE ''
            END,
            CASE
                WHEN age BETWEEN 4 AND 12 THEN 'C'
                WHEN age BETWEEN 13 AND 19 THEN 'T'
                WHEN age BETWEEN 20 AND 34 THEN '1'
                WHEN age BETWEEN 35 AND 49 THEN '2'
                WHEN age >= 50 THEN '3'
            END
        ) AS category

    FROM mst_users_with_age
)

SELECT *
FROM mst_users_with_category;
``` 
### 1-3 연령별 구분의 특징 추출하기

#### 연령대별 행동 분석
- 목적
    - 누가 무엇을 많이 사는가

```sql 
WITH mst_users_with_category AS (
    -- 위 category 코드
    SELECT * FROM ...
)

SELECT
    p.category AS product_category,
    u.category AS user_category,
    COUNT(*) AS purchase_count

FROM action_log p
JOIN mst_users_with_category u
    ON p.user_id = u.user_id

WHERE p.action = 'purchase'

GROUP BY
    p.category,
    u.category

ORDER BY
    p.category,
    u.category;
``` 
- 분석 흐름 
    - 나이 계산
    - 카테고리 생성
    - 분포 확인
    - 행동 데이터 JOIN
    - 인사이트 도출

 
### 1-4 사용자의 방문 빈도 집계하기

#### 방문 빈도 분석
- 목적
    - 유저가 얼마나 자주 서비스 사용하는지 파악
    - CRM / 리텐션 / 충성도 분석 핵심

#### 날짜 추출
- 로그에는 시간까지 있음 -> 날짜만 잘라야 함 

```sql
WITH action_log_with_dt AS (
    SELECT
        *,
        SUBSTRING(stamp, 1, 10) AS dt   -- 날짜만 추출 (YYYY-MM-DD)
    FROM action_log
)
```
#### 유저별 방문일수 계산 
- 핵심 KPI
    - `action_day_count`= 해당 기간동안 몇 일 방문했는지

```sql
WITH action_log_with_dt AS (
    SELECT
        *,
        SUBSTRING(stamp, 1, 10) AS dt
    FROM action_log
),

action_day_count_per_user AS (
    SELECT
        user_id,
        COUNT(DISTINCT dt) AS action_day_count
    FROM action_log_with_dt
    WHERE dt BETWEEN '2016-11-01' AND '2016-11-07'
    GROUP BY user_id
)

SELECT *
FROM action_day_count_per_user;
``` 


### 1-5 벤 다이어그램으로 사용자 액션 집계하기

#### 벤 다이어그램 분석 개념
- 목적
    - 유저가 어떤 기능을 같이 쓰는지 파악

#### 유저별 행동 flag 만들기
- purchase /review / favorite 각각 했는지

```sql
WITH user_action_flag AS (
    SELECT
        user_id,

        SIGN(SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END)) AS has_purchase,
        SIGN(SUM(CASE WHEN action = 'review' THEN 1 ELSE 0 END)) AS has_review,
        SIGN(SUM(CASE WHEN action = 'favorite' THEN 1 ELSE 0 END)) AS has_favorite

    FROM action_log
    GROUP BY user_id
)

SELECT *
FROM user_action_flag;
```

#### 행동 조합별 유저 수 
- 목적
    - 조합별 유저 수 파악
```sql
WITH user_action_flag AS (
    SELECT
        user_id,
        SIGN(SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END)) AS has_purchase,
        SIGN(SUM(CASE WHEN action = 'review' THEN 1 ELSE 0 END)) AS has_review,
        SIGN(SUM(CASE WHEN action = 'favorite' THEN 1 ELSE 0 END)) AS has_favorite
    FROM action_log
    GROUP BY user_id
)

SELECT
    has_purchase,
    has_review,
    has_favorite,
    COUNT(*) AS users

FROM user_action_flag
GROUP BY CUBE(has_purchase, has_review, has_favorite)
ORDER BY has_purchase, has_review, has_favorite;
``` 

#### 결과 보기 좋게 가공
- 0/1 -> 의미있는 텍스트로 변환 
```sql
WITH user_action_flag AS (
    SELECT
        user_id,
        SIGN(SUM(CASE WHEN action = 'purchase' THEN 1 ELSE 0 END)) AS has_purchase,
        SIGN(SUM(CASE WHEN action = 'review' THEN 1 ELSE 0 END)) AS has_review,
        SIGN(SUM(CASE WHEN action = 'favorite' THEN 1 ELSE 0 END)) AS has_favorite
    FROM action_log
    GROUP BY user_id
),

action_venn AS (
    SELECT
        has_purchase,
        has_review,
        has_favorite,
        COUNT(*) AS users
    FROM user_action_flag
    GROUP BY CUBE(has_purchase, has_review, has_favorite)
)

SELECT
    CASE has_purchase
        WHEN 1 THEN 'purchase'
        WHEN 0 THEN 'not purchase'
        ELSE 'any'
    END AS purchase_status,

    CASE has_review
        WHEN 1 THEN 'review'
        WHEN 0 THEN 'not review'
        ELSE 'any'
    END AS review_status,

    CASE has_favorite
        WHEN 1 THEN 'favorite'
        WHEN 0 THEN 'not favorite'
        ELSE 'any'
    END AS favorite_status,

    users,

    -- 비율
    100.0 * users / SUM(users) OVER() AS ratio

FROM action_venn
ORDER BY has_purchase, has_review, has_favorite;
```

### 1-6 Decile 분석을 사용해 사용자를 10단계 그룹으로 나누기

#### Decile 분석
- 목적
    - 유저를 구매력 기준으로 상우ㅏ 10%씩 나누기
- 핵심 흐름
    - 유저별 구매금액 계산
    - 금액 기준 정렬
    - 10개 그룹으로 분할
    - 그룹별 매출 기여도 분석

```sql
WITH user_purchase_amount AS (
    SELECT
        user_id,
        SUM(amount) AS purchase_amount
    FROM action_log
    WHERE action = 'purchase'
    GROUP BY user_id
)

SELECT *
FROM user_purchase_amount;
```

- Decile 생성
```sql
WITH user_purchase_amount AS (
    SELECT
        user_id,
        SUM(amount) AS purchase_amount
    FROM action_log
    WHERE action = 'purchase'
    GROUP BY user_id
),

users_with_decile AS (
    SELECT
        user_id,
        purchase_amount,
        NTILE(10) OVER (ORDER BY purchase_amount DESC) AS decile
    FROM user_purchase_amount
)

SELECT *
FROM users_with_decile;
```
- Decile별 매출 분석
```sql
WITH user_purchase_amount AS (
    SELECT
        user_id,
        SUM(amount) AS purchase_amount
    FROM action_log
    WHERE action = 'purchase'
    GROUP BY user_id
),

users_with_decile AS (
    SELECT
        user_id,
        purchase_amount,
        NTILE(10) OVER (ORDER BY purchase_amount DESC) AS decile
    FROM user_purchase_amount
),

decile_with_purchase AS (
    SELECT
        decile,
        SUM(purchase_amount) AS amount,
        AVG(purchase_amount) AS avg_amount,

        SUM(SUM(purchase_amount)) OVER (ORDER BY decile) AS cumulative_amount,
        SUM(SUM(purchase_amount)) OVER () AS total_amount

    FROM users_with_decile
    GROUP BY decile
)

SELECT *
FROM decile_with_purchase;
``` 
#### 구성비+ 누적 비율
- 상위 몇 %가 매출 %를 차지하는지
```sql
WITH user_purchase_amount AS (
    SELECT
        user_id,
        SUM(amount) AS purchase_amount
    FROM action_log
    WHERE action = 'purchase'
    GROUP BY user_id
),

users_with_decile AS (
    SELECT
        user_id,
        purchase_amount,
        NTILE(10) OVER (ORDER BY purchase_amount DESC) AS decile
    FROM user_purchase_amount
),

decile_with_purchase AS (
    SELECT
        decile,
        SUM(purchase_amount) AS amount,
        AVG(purchase_amount) AS avg_amount,
        SUM(SUM(purchase_amount)) OVER (ORDER BY decile) AS cumulative_amount,
        SUM(SUM(purchase_amount)) OVER () AS total_amount
    FROM users_with_decile
    GROUP BY decile
)

SELECT
    decile,
    amount,
    avg_amount,

    -- 구성비 (%)
    100.0 * amount / total_amount AS total_ratio,

    -- 누적비율 (%)
    100.0 * cumulative_amount / total_amount AS cumulative_ratio

FROM decile_with_purchase
ORDER BY decile;
``` 

### 1-7 RFM 분석으로 사용자를 3가지 관점의 그룹으로 나누기

#### RFM 개념

- 사용자 가치 평가 기준
    - R(Recency): 최근에 언제 구매했냐
    - F(Frequency): 얼마나 자주 구매했냐
    - M(Monetary): 얼마나 많이 썼냐

#### RFM 계산

```sql
WITH purchase_log AS (
    SELECT
        user_id,
        amount,
        SUBSTRING(stamp, 1, 10) AS dt
    FROM action_log
    WHERE action = 'purchase'
),

user_rfm AS (
    SELECT
        user_id,

        MAX(dt) AS recent_date,

        -- 최신 구매로부터 경과일
        DATEDIFF(CURRENT_DATE, MAX(dt)) AS recency,

        COUNT(*) AS frequency,

        SUM(amount) AS monetary

    FROM purchase_log
    GROUP BY user_id
)

SELECT *
FROM user_rfm;
```

#### RFM 점수화
- 기준은 서비스마다 다르게 설정

```sql
WITH user_rfm AS (
    SELECT * FROM ...
),

user_rfm_rank AS (
    SELECT
        user_id,
        recency,
        frequency,
        monetary,

        -- R (작을수록 좋음)
        CASE
            WHEN recency < 14 THEN 5
            WHEN recency < 28 THEN 4
            WHEN recency < 60 THEN 3
            WHEN recency < 90 THEN 2
            ELSE 1
        END AS r,

        -- F (클수록 좋음)
        CASE
            WHEN frequency >= 20 THEN 5
            WHEN frequency >= 10 THEN 4
            WHEN frequency >= 5 THEN 3
            WHEN frequency >= 2 THEN 2
            ELSE 1
        END AS f,

        -- M (클수록 좋음)
        CASE
            WHEN monetary >= 300000 THEN 5
            WHEN monetary >= 100000 THEN 4
            WHEN monetary >= 30000 THEN 3
            WHEN monetary >= 5000 THEN 2
            ELSE 1
        END AS m

    FROM user_rfm
)

SELECT *
FROM user_rfm_rank;
``` 
#### RFM 그룹 집계
- 몇 명이 어떤 그룹인지

```sql
WITH user_rfm_rank AS (
    SELECT * FROM ...
)

SELECT
    r, f, m,
    COUNT(*) AS user_count
FROM user_rfm_rank
GROUP BY r, f, m
ORDER BY r DESC, f DESC, m DESC;
``` 

#### RFM 합 점수
- 단순화

```sql
WITH user_rfm_rank AS (
    SELECT * FROM ...
)

SELECT
    r + f + m AS total_score,
    COUNT(*) AS user_count
FROM user_rfm_rank
GROUP BY total_score
ORDER BY total_score DESC;
``` 

#### 2D 분석(R vs F)

```sql
WITH user_rfm_rank AS (
    SELECT * FROM ...
)

SELECT
    CONCAT('R', r) AS r_group,

    COUNT(CASE WHEN f = 5 THEN 1 END) AS f5,
    COUNT(CASE WHEN f = 4 THEN 1 END) AS f4,
    COUNT(CASE WHEN f = 3 THEN 1 END) AS f3,
    COUNT(CASE WHEN f = 2 THEN 1 END) AS f2,
    COUNT(CASE WHEN f = 1 THEN 1 END) AS f1

FROM user_rfm_rank
GROUP BY r
ORDER BY r DESC;
``` 

#### Decile vs RFM 차이

| 구분      | Decile       | RFM           |
|----------|--------------|--------------|
| 기준      | 금액만 봄     | 행동 전체 봄   |
| 복잡도    | 단순         | 정교         |
| 활용 목적 | 매출 분석     | CRM 분석      |


### 🎉 수고하셨습니다.