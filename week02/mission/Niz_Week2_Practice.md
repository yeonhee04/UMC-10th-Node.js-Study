# Niz_Week2_Practice

### 🔮실습1: SQL 마스터 도전

```sql
[ 집계 함수 ]
"회원별 대여 횟수 상위 5명을 조회하자."

1. 데이터 합치기
FROM member m
JOIN rent r ON m.id=r.member_id
-> 회원 정보(member)와 대여 기록(rent) 데이터 가져오기

2. 그룹으로 묶기 (⭐)
GROUP BY m.id, m.name
-> 1번 과정을 거치면 한 회원이 책을 10번 빌렸을 경우, 그 회원의 이름이 적힌 줄이 10줄이나 생겨남.
   `GROUP BY`는 이렇게 흩어져 있는 줄들을 '회원별로' 묶어주는 역할을 수행함.

3. 계산하고 보여주기
SELECT m.name, COUNT(*) as rent_count
-> `COUNT(*)`는 각 그룹 안에 기록이 몇 줄 들어있는지 개수를 세는 집계 함수임.
   `as rent_count`로 별명을 지정함.

4. 정렬하기
ORDER BY rent_count DESC
-> 대여 횟수(rent_count)를 기준으로 내림차순(DESC) 정렬함.

5.자르기
LIMIT 5;
-> 상위 5명만 출력함.
```

```sql
[ 복합 JOIN ]
"각 회원이 좋아요를 누른 책의 카테고리별 분포를 조회하자."

1. 데이터 합치기
-> `member` 테이블에서 `book_category`로 한 번에 가는 직통 선이 없음.
FROM member m
JOIN book_likes bl ON m.id=bl.member_id
JOIN book b ON bl.book_id=b.id
JOIN book_category bc ON b.book_category_id=bc.id

2. 그룹으로 묶기 (⭐)
-> '사람별 + 카테고리별'로 그룹화하기.
GROUP BY m.id, m.name, bc.id, bc.name;

3. 계산하고 보여주기
-> '사람별 + 카테고리별'로 좋아요 개수를 보여줌.
SELECT m.name, bc.name as category, COUNT(*) as like_count
```

```sql
[ 서브쿼리 ]
"각 회원이 좋아요를 누른 책의 카테고리별 분포를 조회하자."

1. 무작정 모든 조합 만들기 (`CROSS JOIN`)
FROM member m
CROSS JOIN book_category bc
-> `CROSS JOIN`: 모든 경우의 수를 다 곱해서 표를 만듦.

2. 각각의 조합마다 개수 세기 (`SELECT` 안의 서브쿼리)
SELECT m.name, bc.name as category,
( SELECT COUNT(*)
FROM book_likes bl
JOIN book b ON bl.book_id=b.id
WHERE bl.member_id = m.id AND b.book_category_id=bc.id) as like_count
-> "이 회원이 이 카테고리 책에 누른 좋아요 개수"를 `book_likes` 테이블을 거쳐서 계산함.

3. 가짜 조합 버리기 (`WHERE` 안의 서브쿼리)
WHERE (SELECT COUNT(*)
				FROM book_likes bl
				JOIN book b ON bl.book_id=b.id
				WHERE bl.member_id = m.id AND b.book_category_id = bc.id) > 0;
-> 가짜 조합 = 좋아요 개수가 0개인 조합
   "방금 계산한 개수가 0보다 큰(> 0) 진짜 조합만 결과에 남겨라"라고 필터링을 지시함.
```

---

### 💜실습2: 여러 요구 사항에 대응하기

```sql
[ 해시태그로 검색하기 (N:M 관계와 매핑 테이블) ]
"UMC라는 이름을 가진 해시태그가 붙은 책 찾기"

방법 A) 서브쿼리 사용하기
select * from book where id in
	(select book_id from book_hash_tag
			where hash_tag_id  = (select id from hash_tag where name = 'UMC' ));
			1. hash_tag 테이블에서 이름이 'UMC'인 태그의 고유 번호(id) 찾기
	2. 해당 번호의 태그가 붙은 책들의 번호(book_id) 명단을 선정하기
3. book 테이블에서 해당 명단에 있는 번호(id)와 일치하는 책 정보를 모두 가져오기

방법 B) Inner Join 사용하기
select b.*
from book as b
inner join book_hash_tag as bht on b.id = bht.book_id
inner join hash_tag as ht on bht.hash_tag_id = ht.id
where ht.name = 'UMC';
-> 세 개의 테이블(book, book_hash_tag, hash_tag)을 연결고리(각각의 id들)를 기준으로 쭉 이어 붙임.
-> 합쳐진 표 안에서 해시태그 이름이 'UMC'인 줄(where ht.name = 'UMC')만 찾아내어
-> 해당 책의 정보(b.*)를 보여주는 방식
```

```sql
"최신 순 조회"
select * from book order by created_at desc;
-> 책 정보를 모두 가져오되 (select * from book),
-> created_at (생성 시간)을 기준으로 desc (내림차순, 즉 최신 시간이 먼저 오도록) 정렬하라.
```

```sql
"좋아요 개수 순(인기 순) 조회"
select * from book as b
join (select count(*) as like_count
			from book_likes
			group by book_id) as likes on b.id = likes.book_id
order by likes.like_count desc;
1. 통계 표 만들기: 좋아요 기록이 담긴 book_likes 테이블에서
                  책 아이디(book_id)별로 그룹을 묶고 (group by book_id),
                  각 그룹의 개수를 세어 like_count라는 이름을 붙임.
                  -> 이 임시 표를 쿼리에서는 likes라고 부름.
2. 합치기: 원본 책 테이블(book as b)과 방금 만든 통계 표(likes)를 조인(join)하여,
          책 아이디를 기준으로 가로로 이어 붙임 (on b.id = likes.book_id).
          -> 책의 상세 정보 옆에 좋아요 개수가 나란히 위치하게 됨.
3. 정렬하기: 합쳐진 전체 결과물을, like_count 숫자가 큰 순서대로 내림차순 정렬함.
            (order by likes.like_count desc).
```

---

### 💟실습3. 페이지네이션 배워보기

- `LPAD(값, 10, '0')`: 값을 10자리 칸에 넣고, 남는 빈칸은 왼쪽에 '0'으로 꽉 채워라.
- `CONCAT(A, B)`: A와 B를 하나의 문자로 나란히 이어 붙여라.

e.g. 좋아요 23 ➔ `"0000000023"`
아이디 45 ➔ `"0000000045"`
`CONCAT`으로 이어 붙임 ➔ **`"00000000230000000045"`**
