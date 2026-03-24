# Niz_Week2_Mission

## 미션 1: 내가 진행 중, 진행 완료한 미션 모아서 보기 (페이징 포함)

### 1. 화면 분석 및 필요 데이터 추출

- **보상 포인트:** (예: 500P)
- **미션 상태:** (예: 성공, 진행 중/진행 완료)
- **가게 이름:** (예: 가게이름a)
- **미션 내용:** (예: 12,000원 이상의 식사를 하세요!)

### 2. 관련 테이블 파악

- `member_mission`: 사용자 아이디(`member_id`), 미션 상태(`status`), + 정렬 및 페이징의 기준이 될 고유 아이디(`id`) 확인하기.
- `mission`: 미션 보상(`reward`), 미션 내용(`mission_spec`), 연결된 가게 정보(`store_id`) 확인하기.
- `store`: 화면에 보여줄 가게 이름(`name`) 가져오기.

### 3. 쿼리 작성

```sql
SELECT
    s.name AS store_name,
    m.reward,
    m.mission_spec,
    mm.status
FROM member_mission AS mm
JOIN mission AS m ON mm.mission_id = m.id
JOIN store AS s ON m.store_id = s.id
WHERE mm.member_id = {현재 로그인한 사용자의 id}
  AND mm.status = '진행중' /* 탭 선택에 따라 '진행완료'로 문자열이 동적으로 변경됨 */
  AND mm.id < {마지막으로 조회한 member_mission의 id} /* 커서 페이징 조건 */
ORDER BY mm.id DESC
LIMIT 15; /* 한 번에 스크롤로 불러올 개수 */
```

<aside>
💡

1. `member_mission` 테이블을 시작점으로 삼고,
   `mission`과 `store` 테이블의 정보들을 조인하여 가로로 긴 표 만들기.
2. 특정 사용자(`member_id`)의 데이터 중에서,
   현재 사용자가 보고 있는 탭의 상태(`status`)와 일치하는 기록만 골라내기.
3. 중복이나 누락을 방지하기 위해, 사용자가 마지막으로 스크롤해서 본 항목의 `id`보다 작은(더 예전에 수락한) 미션들을 최신순으로 15개씩 잘라서 가져오기.
</aside>

---

## 미션 2: 리뷰 작성하기

### 1. 화면 분석 및 필요 데이터 추출

- **별점 (score):** (예: 5점)
- **리뷰 내용 (body):** (예: "음 너무 맛있어요 포인트도 얻고...")
- **작성자 정보 (member_id):** 닉네임이 보이지만 실제 DB에는 고유 번호로 저장됨.
- **작성일 (created_at):** (예: 2022.05.14)

### 2. 관련 테이블 파악

- `review` : `id`, `member_id`, `store_id`, `body`, `score`, `created_at` 컬럼 확인하기.

### 3. 쿼리 작성 (INSERT 문)

→ 새로운 행(Row)을 추가하기 위해 `INSERT INTO` 구문 사용.

```sql
INSERT INTO review (
    member_id,
    store_id,
    body,
    score,
    created_at
) VALUES (
    {현재 로그인한 사용자의 id},
    {리뷰를 남기는 가게의 id},
    '음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!',
    5.0,
    NOW() /* 데이터베이스의 현재 시간을 자동으로 입력하는 함수 */
);
```

<aside>
💡

1. `INSERT INTO review (...)` 부분에 데이터를 채워 넣을 컬럼들의 이름 순서대로 나열하기.
2. `VALUES (...)` 부분에 앞서 나열한 컬럼 순서에 맞게 실제 데이터 값 넣어주기.
</aside>

---

## **미션 3: 홈 화면 쿼리 (특정 지역에서 도전 가능한 미션 목록)**

### 1. 화면 분석 및 필요 데이터 추출

- **가게 이름:** (예: 반이학생마라탕)
- **미션 내용:** (예: 10,000원 이상의 식사시)
- **보상 포인트:** (예: 500 P 적립)
- **남은 기한:** (예: D-7, DB의 `deadline`을 활용하여 FE에게 토스)

### 2. 관련 테이블 파악

- `region`: 사용자가 선택한 지역 이름(`name`)을 필터링하기 위해 필요.
- `store`: 지역과 미션 연결 역할, 화면에 띄울 가게 이름(`name`) 필요.
- `mission`: 미션에 대한 핵심 정보(`reward`, `deadline`, `mission_spec`) 필요.
- `member_mission` (⭐): 사용자가 이미 도전 중이거나 완료한 미션은 이 목록에서 안 보이게 빼야 함.

### 3. 쿼리 작성 (서브쿼리 + 커서 페이징)

```sql
SELECT
    s.name AS store_name,
    m.mission_spec,
    m.reward,
    m.deadline
FROM mission AS m
JOIN store AS s ON m.store_id = s.id
JOIN region AS r ON s.region_id = r.id
WHERE r.name = '안암동' /* 1. 선택된 지역 필터링 */

  /* 2. 실습 응용: 내가 이미 수락한 미션은 목록에서 제외 (블랙리스트 방식) */
  AND m.id NOT IN (
      SELECT mission_id
      FROM member_mission
      WHERE member_id = {현재 로그인한 사용자의 id}
  )

  /* 3. 커서 기반 페이징 (미션 아이디를 기준으로 최신순 정렬 시) */
  AND m.id < {마지막으로 화면에서 본 미션의 id}
ORDER BY m.id DESC
LIMIT 15;
```

<aside>
💡

1. `mission`, `store`, `region` 세 개의 테이블을 조인하여
   안암동에 있는 모든 미션의 리스트 뽑아내기.
2. 서브쿼리가 먼저 실행되어 "현재 로그인한 사용자가 참여 중인 미션 번호 명단" 작성하기.
   후에 `NOT IN`을 사용해 그 명단에 있는 미션은 1번 리스트에서 빼기.
3. 화면 스크롤이 끊기지 않도록 직전에 본 미션의 고유 번호(`id`)를 커서로 삼아
그 이전 데이터 15개를 최신순으로 가져오기.
</aside>

---

## **미션 4: 마이 페이지 화면 쿼리**

### 1. 화면 분석 및 필요 데이터 추출

- **닉네임:** (예: nickname012)
- **이메일:** (예: dlapdlf@naver.com)
- **휴대폰 번호 (인증 여부):** (예: 미인증)
- **내 포인트:** (예: 2,500)

### 2. 관련 테이블 파악

- `member` : `nickname`, `email`, `phone_num`, `point` 컬럼 확인하기.

### 3. 쿼리 작성 (서브쿼리 + 커서 페이징)

```sql
SELECT
    nickname,
    email,
    phone_num,
    point
FROM member
WHERE id = {현재 로그인한 사용자의 id};
```

<aside>
💡

1. `member` 테이블에서 현재 로그인한 사용자의 `id`와 일치하는 한 줄의 데이터 찾기.
2. 그중에서 화면에 표시될 닉네임, 이메일, 전화번호, 포인트 정보만 골라서 가져오기.

**\* 휴대폰 미인증 처리:** DB에서 `phone_num` 값을 가져왔을 때 그 값이 비어있다면(`NULL`),
FE 쪽에서 '미인증' 글자와 '인증하기' 버튼을 처리하게 끔 토스.

</aside>
