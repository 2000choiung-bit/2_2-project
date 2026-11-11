# 📝 [Troubleshooting] DB 스키마 연동 및 JWT Swagger 인증 과정 기록

## 1. 개요 (Overview)
- **작업 내용**: DB 팀원의 초기화 스크립트(`init.sql`)와 백엔드 JPA 엔티티 연동, 회원가입/로그인 및 JWT 토큰 기반 사이즈 추천 API 검증
- **관련 기술**: Spring Boot, Spring Security, JWT, MySQL, JPA/Hibernate, Swagger UI

---

## 2. 발생한 문제 및 해결 과정 (Trouble Shooting)

### 🔴 문제 1: JPA 엔티티와 DB 초기화 스크립트(`init.sql`) 간 테이블 및 컬럼 불일치
- **원인**: 백엔드에서는 `member_profile` 테이블과 `arm` 컬럼을 기준으로 엔티티를 작성했으나, DB 팀원의 `init.sql`에서는 테이블명이 `fitting_profiles`이고 컬럼도 `gender`, `hip` 등으로 다르게 정의되어 있었음. 또한 회원가입 시 입력받은 `height`, `weight`가 `fitting_profiles`에 자동 저장되지 않아 추천 API 호출 시 프로필 조회가 불가능했음.
- **해결**: `MemberProfile` 엔티티의 `@Table(name = "fitting_profiles")` 및 컬럼 매핑을 `init.sql`과 100% 일치시키고, `UserService`의 회원가입 로직에 `MemberProfileRepository`를 주입해 기본 신체 스펙이 함께 저장되도록 수정함.
- **느낀점**: `Product`와 `ProductSize`는 사전에 구조가 일치해 바로 연동되었지만, `MemberProfile`처럼 테이블명이나 세부 컬럼명이 어긋난 부분은 직접 DDL(`init.sql`)과 엔티티를 1:1로 대조해야 정확히 잡을 수 있다는 것을 배웠다.

### 🔴 문제 2: 회원가입 시 `users.name` 컬럼 `NOT NULL` 제약조건으로 인한 500 에러
- **원인**: 백엔드 회원가입 기획상 `username`(아이디), `password`, `height`, `weight`만 입력받도록 구현했으나, `init.sql`의 `users` 테이블에는 `name VARCHAR(50) NOT NULL` 제약조건이 걸려 있어 `INSERT` 시 DB 예외가 발생함.
- **해결**: 로컬 MySQL에서 `ALTER TABLE users MODIFY COLUMN name VARCHAR(50) NULL;`을 실행해 즉시 테스트를 통과시켰으며, 팀 통합 시 충돌을 방지하기 위해 DB 담당 팀원에게 `init.sql`의 `name` 컬럼 `NULL` 허용 변경을 요청하기로 정리함.
- **느낀점**: 백엔드 API 명세(입력받는 필드)와 DB 테이블의 `NOT NULL` 제약조건이 조금만 달라도 런타임 500 에러로 이어지기 때문에, 스키마 변경 사항이 생길 때마다 팀원 간 즉시 공유하고 동기화하는 과정이 매우 중요하다는 것을 느꼈다.

### 🔴 문제 3: Swagger UI에서 API 테스트 시 `403 Forbidden` 발생 및 인증 버튼 부재
- **원인**: Spring Security에 JWT 인증 필터를 적용한 이후, 로그인으로 발급받은 JWT 토큰을 Swagger 요청 헤더(`Authorization: Bearer ...`)에 실어 보낼 수 있는 `Authorize` 버튼이 `SwaggerConfig`에 설정되어 있지 않았음.
- **해결**: `SwaggerConfig`에 `SecurityScheme(Type.HTTP, scheme = "bearer", bearerFormat = "JWT")`와 `SecurityRequirement`를 추가해 Swagger UI 상단에 `Authorize` 버튼을 활성화하고, 토큰 등록 후 추천 API(`200 OK`) 정상 응답을 확인함.
- **느낀점**: `403 Forbidden` 에러가 떴을 때 당황하지 않고 보안 필터가 정상적으로 미인증 요청을 차단하고 있음을 먼저 파악한 뒤, 테스트 도구(Swagger) 환경에 맞게 인증 스펙을 추가해 원활하게 검증할 수 있었다.
