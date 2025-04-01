# 2-6. 연습문제

**1. 포켓몬 중에 type2가 없는 포켓몬의 수를 작성하는 쿼리 작성**
- 힌트 ~가 없다: col IS NULL

```sql
SELECT 
 COUNT (id) AS cnt
FROM basic.pokemon 
WHERE 
type2 IS NULL
 # WHERE절에서 여러 조건을 연결하고 싶은 경우 => AND
 # () OR ()  
```

**2. type2가 없는 포켓몬의 type1과 type1의 포켓몬 수를 알려주는 쿼리를 작성. 단, type1의 포켓몬 수가 큰 순으로 정렬**

```sql
SELECT
 type1, 
 COUNT (id) AS pokemon_cnt
 # 집계 함수는 GROUP BY와 함께 다님, 집계 기준(컬럼)이 없으면 Count만 o
FROM basic.pokemon 
WHERE 
type2 IS NULL
GROUP BY
 type1 
ORDER BY
 pokemon_cnt DESC
```

**3. type2와와 상관없이 type1의 포켓몬 수를 알 수 있는 쿼리를 작성**
- type2와 상관없이 => 조건이 아님
```sql
SELECT
 SELECT
 type1, 
 COUNT (id) AS pokemon_cnt,
 COUNT (DISTINCT id) AS pokemon_cnt2
FROM basic.pokemon
WHERE 
type2 IS NULL
GROUP BY
 type1 
```

**4. 전설 여부에 따른 포켓몬 수를 알 수 있는 쿼리를 작성**
- 전설 여부 => 조건이 아님
```sql
SELECT 
 is_legendary,
 COUNT (id) AS pokemon_cnt
FROM basic.pokemon
GROUP BY
```
- GROUP BY 1 => SELECT의 첫 컬럼을 의미

**5. 동명이인이 있는 이름은 무엇일까요?**
```SQL
SELECT 
 name,
 COUNT (name) AS trainer_cnt
FROM basic.trainer 
GROUP BY
 name
# 집계 후 조건 => HAVING
HAVING
 trainer_cnt >= 2
```

**6. trainer 테이블에서 "Iris" 트레이너의 정보를 알 수 있는 쿼리를 작성**
```sql
SELECT 
 *
FROM basic.trainer
WHERE
 name = "Iris"
```

**7. trainer 테이블에서 "Iris", "Whittney", "Cynthia" 트레이너의 정보를 알 수 있는 쿼리 작성**
```sql
SELECT
 *
FROM `basic.trainer`
WHERE 
 (name = "Iris")
 OR (name = "Cynthia")
 OR (name = "Whittney")
# => name IN ("Iris", "Cynthia", "Whittney") 이라고 작성해도 동일한 결과
```

**9. 세대 별 포켓몬 수가 얼마나 되는 지 알 수 있는 쿼리**
```sql
SELECT 
 generation,
 COUNT (id) AS pokemon_cnt
FROM basic.pokemon
GROUP BY
 generation
```

**10. type2가 존재하는 포켓몬의 수**
```SQL
SELECT
 COUNT (id) AS pokemon_cnt
FROM basic.pokemon
WHERE 
 type2 IS NOT NULL
```

**11. type2가 있는 포켓몬 중 제일 많은 type**
```sql
SELECT 
 type1,
 COUNT (id) AS pokemon_cnt
FROM basic.pokemon
WHERE
 type2 IS NOT NULL
GROUP BY
 pokemon_cnt DESC
Limit 1 # 1개만 나오게 제한
```

**13. 포켓몬 이름에 '파'가 들어가는 포켓몬**
```sql
SELECT
 kor_name
FROM basic.pokemon
WHERE 
 kor_name LIKE "파%"
 # 컬럼 LIKE "특정단어%"
 # a%: a로 시작하는 단어 / %a: a로 끝나는 단어 / %a%: a가 들어가는 단어
 ```

**17. 트레이너 별로 풀어준 포켓몬 비율이 20%가 넘는 포켓몬 트레이너**
```sql
SELECT
 trainer_id,
 COUNTIF(status = "Released") AS  released_cnt,
 COUNT(pokemon_id) AS pokemon_cnt
 COUNTIF(status = "Released")/COUNT(pokemon_id) AS released_ratio
FROM basic.trainer_pokemon
GROUP BY
 trainer_id
HAVING 
 released_ratio >= 0.2
```

# 2-7. SUMMARY
- 조건(필터링): WHERE / HAVING
- 추출: SELECT
- 변환
- 요약: GROUP BY 
 - 집계함수: AVG, COUNT, COUNTIF, SUM, MAX, MIN
- GROUP BY ALL: select한 모든 컬럼


# 3-2. SQL 쿼리 작성하는 흐름
- 지표 고민: 어떤 문제를 해결하기 위해 데이터가 필요한가?
- 지표 구체화: 구체적인 지표 명시(분자, 분모 표시)
- 지표 탐색: 유사한 문제를 해결한 케이스가 있나 확인 -> 해당 쿼리 리뷰
- 쿼리 작성: 여러 개면 연결 방법(Join) 고민
- 데이터 정합성 확인: 예상한 결과와 동일한지 확인
- 쿼리 가독성: 나중을 위해 깔끔하게 쿼리 작성
- 쿼리 저장: 쿼리는 재사용 되므로 문서로 저장

# 3-3. 쿼리 작성 템플릿과 생산성 도구

**쿼리 작성 템플릿**
- 쿼리를 작성하는 목표, 확인할 지표
- 쿼리 계산 방법
- 데이터의 기간
- 사용할 테이블
- join key
- 데이터 특징
*글로 작성하면 쿼리 작성하기 수월*

**생산성 도구: 템플릿 쉽게 사용하기**
사용하기 전에 자꾸 까먹으니까...생산성 도구 활용!

# 수행 인증
![수행인증](sql_images\KakaoTalk_20250401_191001473.jpg)