<aside>
💡

### 1단계: 사용자 및 기초 설정 (회원가입, 선호도 조사)

→ WF 中 회원가입, 선호 음식 종류 선택

1. **사용자 정보 - [Table: member]**

| **컬럼명**        | **타입**        | **제약 조건**        | **설명**                     |
| ----------------- | --------------- | -------------------- | ---------------------------- |
| **id**            | **bigint**      | **PK**               | 사용자 고유 인덱스           |
| name              | varchar(20)     | Not Null             | 사용자 실명                  |
| gender            | varchar(10)     | Not Null             | 성별 (남성, 여성, 선택안함)  |
| birth_date        | date            | Not Null             | 생년월일                     |
| address           | varchar(100)    | Not Null             | 사용자 주소                  |
| email             | varchar(50)     | Not Null             | 이메일                       |
| phone_num         | varchar(15)     | Not Null             | 전화번호                     |
| nickname          | varchar(20)     | Unique               | 서비스 닉네임                |
| point             | int             | Default 0            | 현재 보유 포인트             |
| **status**        | **varchar(15)** | **Default 'active'** | 회원 상태 (active, inactive) |
| **inactive_date** | **datetime**    |                      | 탈퇴 시점 (Soft Delete용)    |
| **created_at**    | **datetime(6)** | **Not Null**         | 가입일                       |
| **updated_at**    | **datetime(6)** | **Not Null**         | 정보 수정일                  |

1. **음식 카테고리 - [Table: food_category]**

| **컬럼명** | **타입**    | **제약 조건** | **설명**             |
| ---------- | ----------- | ------------- | -------------------- |
| id         | bigint      | PK            | 카테고리 고유 인덱스 |
| name       | varchar(15) | Not Null      | 카테고리명           |

1. **(N:M 매핑) 사용자 선호 음식 - [Table: member_prefer]**

| **컬럼명**  | **타입** | **제약 조건** | **설명**              |
| ----------- | -------- | ------------- | --------------------- |
| id          | bigint   | PK            | 매핑 고유 인덱스      |
| member_id   | bigint   | FK (member)   | 사용자 ID             |
| category_id | bigint   | FK (category) | 선호 음식 카테고리 ID |

### 2단계: 가게 및 미션 핵심 설계 (홈, 미션, 리뷰)

→ 지역별 가게, 미션, 미션 수행 상태 관리

1. **지역 정보 - [Table: region]**

| **컬럼명** | **타입**    | **제약 조건** | **설명**         |
| ---------- | ----------- | ------------- | ---------------- |
| id         | bigint      | PK            | 지역 고유 인덱스 |
| name       | varchar(20) | Not Null      | 지역명           |

1. **가게 정보 - [Table: store]**

| **컬럼명**  | **타입**     | **제약 조건**      | **설명**         |
| ----------- | ------------ | ------------------ | ---------------- |
| id          | bigint       | PK                 | 가게 고유 인덱스 |
| region_id   | bigint       | FK (region)        | 소속 지역 ID     |
| category_id | bigint       | FK (food_category) | 가게 음식 종류   |
| name        | varchar(50)  | Not Null           | 가게 이름        |
| address     | varchar(100) | Not Null           | 가게 상세 주소   |
| score       | float        | Default 0          | 가게 평균 별점   |

1. **미션 정보 - [Table: mission]**

| **컬럼명**   | **타입** | **제약 조건** | **설명**              |
| ------------ | -------- | ------------- | --------------------- |
| id           | bigint   | PK            | 미션 고유 인덱스      |
| store_id     | bigint   | FK (store)    | 미션 대상 가게 ID     |
| reward       | int      | Not Null      | 개별 미션 달성 포인트 |
| deadline     | datetime | Not Null      | 미션 마감 기한        |
| mission_spec | text     | Not Null      | 미션 상세 내용        |

1. **(N:M 매핑) 사용자별 미션 수행 현황 - [Table: member_mission]**

| **컬럼명** | **타입**    | **제약 조건**     | **설명**                      |
| ---------- | ----------- | ----------------- | ----------------------------- |
| id         | bigint      | PK                | 수행 기록 인덱스              |
| member_id  | bigint      | FK (member)       | 사용자 ID                     |
| mission_id | bigint      | FK (mission)      | 미션 ID                       |
| status     | varchar(15) | Default 'ongoing' | 진행 상태 (ongoing, complete) |
| created_at | datetime(6) | Not Null          | 미션 도전 시작일              |
| updated_at | datetime(6) | Not Null          | 상태 변경일                   |

1. **리뷰 정보 - [Table: review]**

| **컬럼명** | **타입**    | **제약 조건** | **설명**          |
| ---------- | ----------- | ------------- | ----------------- |
| id         | bigint      | PK            | 리뷰 인덱스       |
| member_id  | bigint      | FK (member)   | 작성자 ID         |
| store_id   | bigint      | FK (store)    | 리뷰 대상 가게 ID |
| body       | text        | Not Null      | 리뷰 내용         |
| score      | float       | Not Null      | 부여 별점         |
| created_at | datetime(6) | Not Null      | 작성일            |

</aside>
