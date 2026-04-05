# SQL_MASTER 5주차 정규과제

📌SQL MASTER 정규과제는 매주 정해진 분량의 『*데이터 분석을 위한 SQL 레시피*』 를 읽고 학습하는 것입니다. 이번 주는 아래의 **SQL_MASTER_5th_TIL**에 나열된 분량을 읽고 공부하시면 됩니다.

아래 실습을 수행하며 학습 내용을 직접 적용해보세요. 단순히 결과를 재현하는 것이 아니라, SQL을 직접 작성하는 과정에서 개념을 스스로 정리하는 것이 중요합니다.

필요한 경우 교재와 추가 자료를 참고하여 이해를 보완하시기 바랍니다.

## SQL_MASTER_5th_TIL

### 5장 사용자를 파악하기 위한 데이터 추출
#### 2. 시계열에 따른 사용자 전체의 상태 변화 찾기
#### 3. 시계열에 따른 사용자의 개별적인 행동 분석하기 


## Study Schedule

| 주차  | 공부 범위     | 완료 여부 |
| ----- | ------------- | --------- |
| 1주차 | p.20~50    | ✅         |
| 2주차 | p.52~136   | ✅         |
| 3주차 | p.138~184  | ✅         |
| 4주차 | p.186~232 | ✅         |
| 5주차 | p.233~321 | ✅         |
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

## 2. 시계열에 따른 사용자 전체의 상태 변화 찾기

- 분석 목적
    - 사용자 상태를 시간 흐름에 따라
        - 신규 유입
        - 활성 사용자
        - 이탈 사용자
- 서비스 성장 / 유지 / 이탈 구조 이해


### 2-1 등록 수의 추이와 경향 보기



```sql
SELECT
    register_date,
    COUNT(DISTINCT user_id) AS register_count
FROM mst_users
GROUP BY register_date
ORDER BY register_date;
```
- 유입 트렌드 확인
- 마케팅 효과 분석 가능

#### 월별 등록 수 + 증감률
```sql
WITH mst_users_with_year_month AS (
    SELECT
        substring(register_date, 1, 7) AS year_month
    FROM mst_users
)

SELECT
    year_month,
    COUNT(DISTINCT user_id) AS register_count,

    LAG(COUNT(DISTINCT user_id))
    OVER(ORDER BY year_month) AS last_month_count,

    1.0 * COUNT(DISTINCT user_id)
    / LAG(COUNT(DISTINCT user_id)) OVER(ORDER BY year_month)
    AS month_over_month_ratio

FROM mst_users_with_year_month
GROUP BY year_month
ORDER BY year_month;
``` 
- `LAG()`: 이전 시점 비교

### 2-2 지속률과 정착률 산출하기

#### 지속률(Retention)
- 정의
    - 특정 날짜 기준 이후 계속 사용한 비율
```sql
WITH action_log_with_mst_users AS (
    SELECT
        u.user_id,
        u.register_date,
        CAST(a.stamp AS DATE) AS action_date,
        MAX(CAST(a.stamp AS DATE)) OVER() AS latest_date,
        DATE(CAST(u.register_date AS DATE), '+1 day') AS next_day_1
    FROM mst_users u
    LEFT JOIN action_log a
        ON u.user_id = a.user_id
),

user_action_flag AS (
    SELECT
        user_id,
        register_date,
        SIGN(SUM(
            CASE
                WHEN next_day_1 <= latest_date THEN
                    CASE WHEN next_day_1 = action_date THEN 1 ELSE 0 END
            END
        )) AS next_1_day_action
    FROM action_log_with_mst_users
    GROUP BY user_id, register_date
)

SELECT
    register_date,
    AVG(100.0 * next_1_day_action) AS repeat_rate_1_day
FROM user_action_flag
GROUP BY register_date
ORDER BY register_date;
``` 
#### 정착률
- 정의
    - 특정 기간동안 1번이라도 사용했는지

```sql
# 기간 정의
WITH repeat_interval(index_name, begin_date, end_date) AS (
    VALUES
        ('7 day retention', 1, 7),
        ('14 day retention', 8, 14),
        ('21 day retention', 15, 21),
        ('28 day retention', 22, 28)
)

SELECT *
FROM repeat_interval;

# 정착률 계산
WITH repeat_interval AS (
    SELECT *
    FROM (VALUES
        ('7 day retention', 1, 7),
        ('14 day retention', 8, 14),
        ('21 day retention', 15, 21),
        ('28 day retention', 22, 28)
    ) AS t(index_name, begin_date, end_date)
),

action_log_with_index_date AS (
    SELECT
        u.user_id,
        u.register_date,
        r.index_name,
        DATE(CAST(u.register_date AS DATE), '+' || r.begin_date || ' day') AS index_begin_date,
        DATE(CAST(u.register_date AS DATE), '+' || r.end_date || ' day') AS index_end_date,
        CAST(a.stamp AS DATE) AS action_date
    FROM mst_users u
    LEFT JOIN action_log a
        ON u.user_id = a.user_id
    CROSS JOIN repeat_interval r
),

user_action_flag AS (
    SELECT
        user_id,
        register_date,
        index_name,
        SIGN(SUM(
            CASE
                WHEN action_date BETWEEN index_begin_date AND index_end_date
                THEN 1 ELSE 0
            END
        )) AS index_date_action
    FROM action_log_with_index_date
    GROUP BY user_id, register_date, index_name
)

SELECT
    register_date,
    index_name,
    AVG(100.0 * index_date_action) AS index_rate
FROM user_action_flag
GROUP BY register_date, index_name
ORDER BY register_date, index_name;
```


### 2-3 지속과 정착에 영향을 주는 액션 집계하기 
- 목적
    - 어떤 행동이 재방문(Retention)에 영향을 주는지 분석
    - 단순 방문이 아니라 행동 기반 인사이트 도출

#### 액션 리스트 생성
```sql
WITH mst_actions AS (
    SELECT 'view' AS action
    UNION ALL SELECT 'comment'
    UNION ALL SELECT 'follow'
)
``` 
#### 사용자x액션 전체 조합 생성 
```sql
WITH mst_user_actions AS (
    SELECT
        u.user_id,
        u.register_date,
        a.action
    FROM mst_users u
    CROSS JOIN mst_actions a
)

SELECT *
FROM mst_user_actions
ORDER BY user_id, action;
```
- `CROSS JOIN`: 모든 경우의 수 생성

#### 액션 수행 여부 flag 생성
```sql
WITH register_action_flag AS (
    SELECT DISTINCT
        m.user_id,
        m.register_date,
        m.action,

        CASE
            WHEN a.action IS NOT NULL THEN 1
            ELSE 0
        END AS do_action,

        f.index_name,
        f.index_date_action

    FROM mst_user_actions m
    LEFT JOIN action_log a
        ON m.user_id = a.user_id
        AND m.action = a.action
        AND CAST(m.register_date AS DATE) = CAST(a.stamp AS DATE)

    LEFT JOIN user_action_flag f
        ON m.user_id = f.user_id
)

SELECT *
FROM register_action_flag
ORDER BY user_id, action;
``` 

#### 액션별 유지율 비교
```sql
SELECT
    action,
    COUNT(1) AS users,

    AVG(100.0 * do_action) AS usage_rate,

    index_name,

    AVG(CASE WHEN do_action = 1 THEN 100.0 * index_date_action END) AS do_action_idx_rate,

    AVG(CASE WHEN do_action = 0 THEN 100.0 * index_date_action END) AS no_action_idx_rate

FROM register_action_flag
GROUP BY index_name, action
ORDER BY index_name, action;
```  

 
### 2-4 액션 수에 따른 정착률 집계하기 

- 목적
    - 행동을 했냐가 아니라 얼마나 많이 했냐

#### 액션 구간 테이블 생성
```sql
WITH mst_action_bucket(action, min_count, max_count) AS (
    VALUES
        ('comment', 0, 0),
        ('comment', 1, 5),
        ('comment', 6, 10),
        ('comment', 11, 9999),

        ('follow', 0, 0),
        ('follow', 1, 5),
        ('follow', 6, 10),
        ('follow', 11, 9999)
)
```
#### 사용자x액션 구간 생성
```sql 
WITH mst_user_action_bucket AS (
    SELECT
        u.user_id,
        u.register_date,
        b.action,
        b.min_count,
        b.max_count
    FROM mst_users u
    CROSS JOIN mst_action_bucket b
)

SELECT *
FROM mst_user_action_bucket
ORDER BY user_id, action, min_count;
``` 

#### 구간별 행동 횟수 계산 + 정착 여부
```sql
WITH register_action_flag AS (
    SELECT
        m.user_id,
        m.action,
        m.min_count,
        m.max_count,

        COUNT(a.action) AS action_count,

        CASE
            WHEN COUNT(a.action) BETWEEN m.min_count AND m.max_count THEN 1
            ELSE 0
        END AS achieve,

        f.index_name,
        f.index_date_action

    FROM mst_user_action_bucket m
    LEFT JOIN action_log a
        ON m.user_id = a.user_id
        AND m.action = a.action

    LEFT JOIN user_action_flag f
        ON m.user_id = f.user_id

    GROUP BY
        m.user_id, m.action, m.min_count, m.max_count,
        f.index_name, f.index_date_action
)

SELECT *
FROM register_action_flag
ORDER BY user_id, action, min_count;
``` 
#### 구간별 정착률
```sql
SELECT
    action,

    min_count,
    max_count,

    CONCAT(min_count, '~', max_count) AS count_range,

    SUM(CASE WHEN achieve = 1 THEN 1 ELSE 0 END) AS achieve_users,

    index_name,

    AVG(CASE WHEN achieve = 1 THEN 100.0 * index_date_action END) AS achieve_index_rate

FROM register_action_flag
GROUP BY action, index_name, min_count, max_count
ORDER BY action, min_count;
``` 

### 2-5 사용 일수에 따른 정착률 집계하기 

- 목적
    - 단순 행동이 아니라 얼마나 자주 사용했는지가 retention에 미치는 영향 분석
- 분석 구조 
    - 로그 데이터 -> 날짜 단위 변환 -> 사용 일수 계산 -> retention 결합 -> 그룹별 비교


### 2-6 사용자의 잔존율 집계하기 

#### 사용자 잔존율(Cohort Retention)
- 목적
    - 사용자 가입 시점별 유지율 분석 
    - 서비스 문제 구간 파악

#### 월 interval 생성
```sql
WITH mst_intervals(interval_month) AS (
    VALUES (1),(2),(3),(4),(5),(6),(7),(8),(9),(10),(11),(12)
)

SELECT *
FROM mst_intervals;
```

#### 사용자 x 기간 생성
```sql
WITH mst_users_with_index_month AS (
    SELECT
        u.user_id,
        u.register_date,

        substring(u.register_date, 1, 7) AS register_month,

        substring(
            DATE(CAST(u.register_date AS DATE), '+' || i.interval_month || ' month'),
            1, 7
        ) AS index_month

    FROM mst_users u
    CROSS JOIN mst_intervals i
)
```

#### 활동 로그 -> 월 단위 변환
```sql
WITH action_log_in_month AS (
    SELECT DISTINCT
        user_id,
        substring(stamp, 1, 7) AS action_month
    FROM action_log
)
```

#### Cohort Retention 계산
```sql
SELECT
    u.register_month,
    u.index_month,

    COUNT(DISTINCT u.user_id) AS users,

    AVG(
        CASE
            WHEN a.action_month IS NOT NULL THEN 100.0
            ELSE 0.0
        END
    ) AS retention_rate

FROM mst_users_with_index_month u
LEFT JOIN action_log_in_month a
    ON u.user_id = a.user_id
    AND u.index_month = a.action_month

GROUP BY u.register_month, u.index_month
ORDER BY u.register_month, u.index_month;
```

### 2-7 방문 빈도를 기반으로 사용자 속성을 정의하고 집계하기

#### 사용자 유형 분류
- 유형
    - 신규: 이번 달 첫 방문
    - 유지: 지난달 -> 이번달
    - 복귀: 과거 -> 다시 방문

#### 월별 사용자 로그 생성
```sql
WITH monthly_user_action AS (
    SELECT DISTINCT
        u.user_id,

        substring(u.register_date, 1, 7) AS register_month,
        substring(a.stamp, 1, 7) AS action_month,

        substring(
            DATE(CAST(a.stamp AS DATE), '-1 month'),
            1, 7
        ) AS action_month_priv

    FROM mst_users u
    JOIN action_log a
        ON u.user_id = a.user_id
)
```

#### 사용자 타입 분류
```sql
WITH monthly_user_with_type AS (
    SELECT
        action_month,
        user_id,

        CASE
            WHEN register_month = action_month THEN 'new_user'

            WHEN action_month_priv =
                 LAG(action_month)
                 OVER(PARTITION BY user_id ORDER BY action_month)
            THEN 'repeat_user'

            ELSE 'come_back_user'
        END AS c

    FROM monthly_user_action
)
```

#### MAU + 사용자 구성
```sql
SELECT
    action_month,

    COUNT(user_id) AS mau,

    COUNT(CASE WHEN c = 'new_user' THEN 1 END) AS new_users,
    COUNT(CASE WHEN c = 'repeat_user' THEN 1 END) AS repeat_users,
    COUNT(CASE WHEN c = 'come_back_user' THEN 1 END) AS come_back_users

FROM monthly_user_with_type
GROUP BY action_month
ORDER BY action_month;
``` 


### 2-8 방문 종류를 기반으로 성장지수 집계하기 

#### 성장지수 (Growth Index)
- 목적
    - 서비스가 실제로 성장 중인지 판단
    - 단순 MAU가 아니라 유저 상태 변화 기반

#### 사용자 데이터 생성
```sql
# 날짜별 사용자 로그 생성
WITH unique_action_log AS (
    SELECT DISTINCT
        user_id,
        substring(stamp, 1, 10) AS action_date
    FROM action_log
)

# 날짜 테이블 생성
WITH mst_calendar AS (
    SELECT '2016-10-01' AS dt
    UNION ALL SELECT '2016-10-02'
    UNION ALL SELECT '2016-10-03'
    UNION ALL SELECT '2016-10-04'
)

# 사용자x날짜 확장
WITH target_date_with_user AS (
    SELECT
        c.dt AS target_date,
        u.user_id,
        u.register_date,
        u.withdraw_date
    FROM mst_users u
    CROSS JOIN mst_calendar c
)

# 사용자 상태 정의
WITH user_status_log AS (
    SELECT
        u.target_date,
        u.user_id,

        CASE WHEN u.register_date = u.target_date THEN 1 ELSE 0 END AS is_new,
        CASE WHEN u.withdraw_date = u.target_date THEN 1 ELSE 0 END AS is_exit,

        CASE WHEN a.action_date IS NOT NULL THEN 1 ELSE 0 END AS is_access,

        LAG(CASE WHEN a.action_date IS NOT NULL THEN 1 ELSE 0 END)
        OVER (PARTITION BY u.user_id ORDER BY u.target_date) AS was_access

    FROM target_date_with_user u
    LEFT JOIN unique_action_log a
        ON u.user_id = a.user_id
        AND u.target_date = a.action_date
)
```

#### 성장지수 계산
```sql
WITH user_growth_index AS (
    SELECT
        target_date,

        CASE
            WHEN is_new = 1 THEN 'signup'
            WHEN is_exit = 1 THEN 'exit'

            WHEN was_access = 0 AND is_access = 1 THEN 'reactivation'
            WHEN was_access = 1 AND is_access = 0 THEN 'deactivation'

            ELSE NULL
        END AS growth_index

    FROM user_status_log
)

SELECT
    target_date,

    SUM(CASE WHEN growth_index = 'signup' THEN 1 ELSE 0 END) AS signup,
    SUM(CASE WHEN growth_index = 'reactivation' THEN 1 ELSE 0 END) AS reactivation,
    SUM(CASE WHEN growth_index = 'deactivation' THEN 1 ELSE 0 END) AS deactivation,
    SUM(CASE WHEN growth_index = 'exit' THEN 1 ELSE 0 END) AS exit,

    SUM(
        CASE
            WHEN growth_index IN ('signup','reactivation') THEN 1
            WHEN growth_index IN ('deactivation','exit') THEN -1
            ELSE 0
        END
    ) AS growth_index

FROM user_growth_index
GROUP BY target_date
ORDER BY target_date;
``` 


### 2-9 지표 개선 방법 익히기 

#### 지표 개선 방법
- 목표
    - 어떤 행동이 지표를 개선하는가?
- 기본 접근 방석
    - 지표 정의
        - retention
        - conversion
        - MAU
    - 행동 기준 분리
        - *댓글 쓴 사용자 vs 안 쓴 사용자*
        - *구매한 사용자 vs 미구매 사용자*
    - 지표 비교 


## 3. 시계열에 따른 사용자의 개별적인 행동 분석하기 

### 3-1 사용자의 액션 간격 집계하기

- 개념
    - 특정 이벤트 간 시간 차이
        - *회원가입 -> 구매*
        - *조회 -> 구매*
        - *요청 -> 주문*

#### 같은 레코드 내 날짜 차이

```sql
SELECT
    reservation_id,
    register_date,
    visit_date,

    visit_date::date - register_date::date AS lead_time

FROM reservations;
```

#### 여러 테이블 간 lead time
```sql
SELECT
    r.user_id,
    r.product_id,

    e.estimate_date - r.request_date AS estimate_lead_time,
    o.order_date - e.estimate_date AS order_lead_time,

    o.order_date - r.request_date AS total_lead_time

FROM requests r
LEFT JOIN estimates e
    ON r.user_id = e.user_id
    AND r.product_id = e.product_id

LEFT JOIN orders o
    ON r.user_id = o.user_id
    AND r.product_id = o.product_id;
```

### 3-2 카트 추가 후에 구매했는지 파악하기 

#### 장바구니 -> 구매 전환 분석 
```sql
# 상품 단위로 분해
SELECT
    user_id,
    action,
    product_id,
    stamp
FROM action_log;

# 장바구니 -> 구매 시간 계산
WITH action_time_stats AS (
    SELECT
        user_id,
        product_id,

        MIN(CASE WHEN action = 'add_cart' THEN stamp END) AS add_cart_time,
        MIN(CASE WHEN action = 'purchase' THEN stamp END) AS purchase_time,

        MIN(CASE WHEN action = 'purchase' THEN stamp END)
        - MIN(CASE WHEN action = 'add_cart' THEN stamp END) AS lead_time

    FROM action_log
    GROUP BY user_id, product_id
)

SELECT *
FROM action_time_stats;

# 시간 구간별 구매 여부
SELECT
    dt,

    COUNT(*) AS add_cart,

    SUM(CASE WHEN lead_time <= 3600 THEN 1 ELSE 0 END) AS purchase_1_hour,
    SUM(CASE WHEN lead_time <= 6*3600 THEN 1 ELSE 0 END) AS purchase_6_hours,
    SUM(CASE WHEN lead_time <= 24*3600 THEN 1 ELSE 0 END) AS purchase_24_hours,
    SUM(CASE WHEN lead_time <= 48*3600 THEN 1 ELSE 0 END) AS purchase_48_hours,

    SUM(CASE WHEN lead_time IS NULL OR lead_time > 48*3600 THEN 1 ELSE 0 END) AS not_purchase

FROM action_time_stats
GROUP BY dt;
```


### 3-3 등록으로부터의 매출을 날짜별로 집계하기 
- 목적
    - 사용자 가입 시점 기준 매출 추척
    - 마케팅 효율/유저 가치 평가
- 핵심 개념
    - Cohort Revenue
- 분석 구조 
    - 가입일 기준 -> 기간 설정 -> 구매 로그 매핑 -> 매출 집계
```sql
# 기간 정의
WITH index_intervals(index_name, interval_begin, interval_end) AS (
    VALUES
        ('30 day sales', 0, 30),
        ('45 day sales', 0, 45),
        ('60 day sales', 0, 60)
)

# 사용자 기준 날짜 설정
WITH mst_users_with_base_date AS (
    SELECT
        user_id,
        register_date AS base_date
    FROM mst_users
)

# 구매로그 + 기간 매핑
WITH purchase_log_with_index_date AS (
    SELECT
        u.user_id,
        u.base_date,

        CAST(a.stamp AS DATE) AS action_date,

        i.index_name,

        DATE(CAST(u.base_date AS DATE), '+' || i.interval_begin || ' day') AS index_begin_date,
        DATE(CAST(u.base_date AS DATE), '+' || i.interval_end || ' day') AS index_end_date,

        a.amount

    FROM mst_users_with_base_date u
    LEFT JOIN action_log a
        ON u.user_id = a.user_id
        AND a.action = 'purchase'

    CROSS JOIN index_intervals i
)

# 기간 내 매출 계산
WITH user_purchase_amount AS (
    SELECT
        user_id,
        index_name,

        SUM(
            CASE
                WHEN action_date BETWEEN index_begin_date AND index_end_date
                THEN amount
                ELSE 0
            END
        ) AS index_date_amount

    FROM purchase_log_with_index_date
    GROUP BY user_id, index_name
)

# 최종 매출 집계
SELECT
    index_name,

    COUNT(user_id) AS users,

    COUNT(CASE WHEN index_date_amount > 0 THEN 1 END) AS purchase_users,

    SUM(index_date_amount) AS total_amount,

    AVG(index_date_amount) AS avg_amount

FROM user_purchase_amount
GROUP BY index_name
ORDER BY index_name;
```

#### 핵심 지표 연결 
- **ARPU**
```
ARPU = total_amount / users
```

- **ARPPU**
```
ARPPU = total_amount / purchase_users
```

- **LTV**
```
LTV = ARPU × retention × 기간
``` 



### 🎉 수고하셨습니다.