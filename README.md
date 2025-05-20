# 이벤트 및 보상 API 시스템

NestJS + MongoDB + Docker 기반의 **이벤트 및 보상 관리 시스템**입니다.  
이 시스템은 이벤트 등록/조회, 보상 요청/승인, 사용자 인증 및 역할 기반 접근 제어를 마이크로서비스로 분리하여 제공합니다.

---

## 주요 기능

| 서비스         | 주요 기능                                                              |
|----------------|------------------------------------------------------------------------|
| Auth 서비스    | 회원가입, 로그인, 사용자 정보 조회, 역할 기반 인증                    |
| Event 서비스   | 이벤트 등록, 이벤트 리스트 조회                                        |
| Reward 서비스  | 보상 요청, 보상 승인, 유저별 요청 이력 조회, 지급 대상 유저 목록 조회 |
| Gateway 서비스 | 모든 서비스의 API Gateway 역할 (Proxy)                                |

---

## 실행 방법

### 1. 프로젝트 클론

```bash
git clone 
cd event-api

### 2. Docker로 실행

```bash
docker-compose up --build
```

### 3. Swagger API 문서 접속

| 서비스     | 주소                                                     |
| ------- | ------------------------------------------------------ |
| Gateway | [http://localhost:3000/api](http://localhost:3000/api) |
| Auth    | [http://localhost:3001/api](http://localhost:3001/api) |
| Event   | [http://localhost:3002/api](http://localhost:3002/api) |

---

## API 목록

### Auth 서비스 (`localhost:3001`)

| Method | 경로               | 설명             |
| ------ | ---------------- | -------------- |
| POST   | /auth/register   | 회원가입           |
| POST   | /auth/login      | 로그인(JWT 발급)    |
| GET    | /auth/me         | 로그인한 사용자 정보 조회 |
| GET    | /auth/admin-only | 관리자 전용 데이터 확인  |

### Event 서비스 (`localhost:3002`)

| Method | 경로            | 설명         |
| ------ | ------------- | ---------- |
| POST   | /event/create | 이벤트 등록     |
| GET    | /event/list   | 이벤트 리스트 조회 |

### Reward 서비스 (`localhost:3002`)

| Method | 경로                   | 설명                           |
| ------ | -------------------- | ---------------------------- |
| POST   | /reward/request      | 보상 요청 (자동 검증 및 자동 승인 포함)     |
| PATCH  | /reward/approve/\:id | 보상 수동 승인 (관리자 또는 운영자만 가능)    |
| GET    | /reward/history      | 보상 요청 이력 조회 (userId 기반 필터)   |
| GET    | /reward-target/list  | 특정 이벤트에 대해 보상을 받을 수 있는 유저 조회 |

### Gateway API (`localhost:3000`)

> 모든 API를 Gateway에서 프록시 방식으로 호출 가능

| Method | 경로                           | 설명         |
| ------ | ---------------------------- | ---------- |
| GET    | /gateway/event/list          | 이벤트 리스트 조회 |
| POST   | /gateway/event/create        | 이벤트 등록     |
| POST   | /gateway/reward/request      | 보상 요청      |
| PATCH  | /gateway/reward/approve/\:id | 보상 승인      |
| POST   | /gateway/auth/login          | 로그인        |
| POST   | /gateway/auth/register       | 회원가입       |

---

## 테스트용 유저 등록

회원가입 API를 통해 직접 유저를 등록할 수 있습니다:

```http
POST /auth/register
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "1234",
  "role": "ADMIN"
}
```

> role 값은 `USER`, `OPERATOR`, `AUDITOR`, `ADMIN` 중 선택 가능
> 관리자/운영자만 `/reward/approve/:id` API 접근 가능

---

## 폴더 구조

```
apps/
  ├── auth/       # 인증 서비스
  ├── event/      # 이벤트 서비스
  ├── gateway/    # API Gateway
libs/
  ├── common/     # 공통 모듈 (decorators, guards 등)
  └── interfaces/ # DTO, 인터페이스
```

---

## docker-compose 구성

```yaml
version: '3.8'
services:
  mongo:
    image: mongo
    ports:
      - "27017:27017"

  auth:
    build: .
    command: npm run start:auth
    ports:
      - "3001:3001"
    depends_on:
      - mongo

  event:
    build: .
    command: npm run start:event
    ports:
      - "3002:3002"
    depends_on:
      - mongo

  gateway:
    build: .
    command: npm run start:gateway
    ports:
      - "3000:3000"
    depends_on:
      - auth
      - event
```

---

## 개발 시 중점적으로 고려한 부분

* **역할 기반 권한 제어**: `@Roles` 데코레이터 + Guard 조합으로 ADMIN/OPERATOR만 승인 가능
* **보상 중복 방지 처리**: 동일 이벤트에 대해 한 번만 보상 요청 가능
* **자동 보상 처리**: 이벤트 유효성 검사 통과 시 자동으로 SUCCESS 처리 및 콘솔 로그 출력
* **마이크로서비스 구조**: Auth / Event / Reward / Gateway 완전 분리 설계
* **Docker 기반 실행 환경 구성**: `docker-compose` 단일 명령어로 전체 실행
* **Swagger 연동**: 모든 서비스에 Swagger 문서 자동 생성

---

## 향후 개선 방향

* Kafka 기반 메시지 큐 연동 (비동기 보상 처리)
* Jenkins 기반 배치 보상 재처리 스케줄링
* Redis 캐시를 활용한 보상 중복 확인 최적화
* Test 코드 작성 및 CI 자동화

---
