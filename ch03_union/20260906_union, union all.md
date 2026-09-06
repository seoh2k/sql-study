# union, union all

생성일: 2026년 9월 6일 오전 9:30
날짜: 2026년 9월 6일
태그: union, unionall

## UNION

```sql
-- 본 실습을 위한 탈퇴 고객 테이블 생성 DROP TABLE IF EXISTS retired_users; CREATE TABLE retired_users (
      id BIGINT PRIMARY KEY,
      name VARCHAR(255) NOT NULL,
      email VARCHAR(255) NOT NULL,
      retired_date DATE NOT NULL
);
-- 탈퇴 고객 데이터 입력
INSERT INTO retired_users (id, name, email, retired_date) VALUES 
(1, '션', 'sean@example.com', '2024-12-31'),
(7, '아이작 뉴턴', 'newton@example.com', '2025-01-10');

-- union 사용 시 중복 제거
select name, email from users
union
select name, email from retired_users;
```

## UNION ALL

```sql
-- union all 사용 시 중복 허용, 빠름
select u.name, u.email
from users u
join orders o on u.user_id = o.user_id
join products p on o.product_id = p.product_id
where p.category = '전자기기'

union all 

select name, email
from users
where address like '서울%';
```

## UNION 정렬

```sql
-- union 정렬: 전체 union 연산의 가장 마지막에 한 번만 사용
select name, email from users
union
select name, email from retired_users
order by name; -- 최종 결과에 대한 정렬

-- 별칭 사용하기
select name, email, created_at as event_date from users
union
select name, email, retired_date as event_date from retired_users
order by event_date;
```

## UNION 문제풀이

```sql
-- 문제1: 전체 고객 목록 조회하기
-- [문제] 우리 쇼핑몰의 모든 고객(활동 고객과 탈퇴 고객)의 이름과 이메일을 중복 없이 조회하여 하나의 목록으로 만들어라.
select name, email from users
union
select name, email from retired_users;

-- 문제2: 특별 이벤트 대상자 목록 만들기 (중복 포함)
-- [문제] 마케팅팀에서 두 그룹의 고객에게 이벤트를 진행하려고 한다. 
-- 1. '전자기기' 카테고리 상품을 한 번이라도 구매한 고객
-- 2. 한번의 주문으로 상품을 2개이상 구매한 고객
-- 두 그룹에 모두 해당하는 고객도 있을 수 있다. 
-- 성능을 고려하여, 두 그룹의 목록을 **중복 제거 없이** 모두 합쳐서 조회해라. 
-- 컬럼 별칭은 `고객명` , `이메일` 로 지정한다.
select u.name as 고객명, u.email as 이메일 
from users u
join orders o on u.user_id = o.user_id
join products p on o.product_id = p.product_id
where p.category = '전자기기'

union all

select distinct u.name as 고객명, u.email as 이메일 
from users u
join orders o on u.user_id = o.user_id
where o.quantity > 1;

-- 문제3: 회사 주요 이벤트 타임라인 만들기
-- [문제]
-- 회사의 주요 이벤트 타임라인을 만들려고 한다. 
-- '고객 가입' 이벤트와 '상품 주문' 이벤트를 시간 순서대로 정렬하여 조회 해라.
-- `users` 테이블의 `created_at` 은 '고객 가입' 이벤트로 간주한다.
-- `orders` 테이블의 `order_date` 는 '상품 주문' 이벤트로 간주한다.
-- 결과는 `이벤트_날짜` , `이벤트_종류` , `상세_내용` 컬럼으로 구성한다.
-- `상세_내용` 에는 고객 가입 시 고객의 이름, 상품 주문 시 상품의 이름을 표시한다. 
-- 최신 이벤트가 가장 위에 오도록 내림차순으로 정렬해라.
select 
	created_at as 이벤트_날짜, 
    '고객 가입' as 이벤트_종류, 
    name as 상세_내용 
from users

union all

select 
	o.order_date as 이벤트_날짜, 
    '상품 주문' as 이벤트_종류, 
    p.name as 상세_내용  
from orders o
join products p on o.product_id = p.product_id

order by 이벤트_날짜 desc;

-- 문제4: 회사 전체 인명록 만들기
-- [문제]
-- 회사의 모든 관련 인물( `users` 의 고객과 `employees` 의 직원)을 통합한 인명록을 만들어라.
-- 결과에는 `이름` , `역할` ('고객' 또는 '직원'), `이메일` 컬럼이 포함되어야 한다. 
-- 고객의 연락처는 `email` 컬럼을 사용한다.
-- 직원의 연락처는 `name` 컬럼 뒤에 `'@my-shop.com'` 을 붙여서 생성한다. 
-- 최종 결과는 이름(가나다/알파벳 순)으로 오름차순 정렬해라.
select 
	name as 이름, 
    '고객' as 역할, 
    email as 이메일 
from users

union all

select 
	name as 이름, 
    '직원' as 역할, 
    concat(name, '@my-shop.com') as 이메일 
from employees

order by 이름;
```