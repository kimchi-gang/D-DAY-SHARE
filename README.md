# D-DAY-SHARE
> “일정을 공유하고, 나에게 중요한 날의 D-day를 매일 카카오톡으로 알림 받자”

---

# 📘 프로젝트 개발 문서

---

## 1. 프로젝트 개요

### 📌 프로젝트 명
**D-DAY SHARE**

### 🧩 프로젝트 구조
- **Backend:** Java 17 + Spring Boot 3.x
- **App:** Android Native (Kotlin + Jetpack Compose)
- **DB:** MySQL
- **Push:** FCM
- **알림:** 카카오 알림톡

---

### 📖 프로젝트 설명

D-DAY SHARE는 스터디, 팀, 가족 등 **그룹 단위 사용자**가  
각자의 중요한 일정을 이름별로 공유하고,  
개인 설정에 따라 D-day 알림을 받을 수 있는 서비스입니다.

기존 캘린더 서비스는  
- 일정 공유는 가능하지만  
- 작성자 구분이 명확하지 않고  
- D-day 중심 관리 및 개인화 알림이 부족합니다.

본 서비스는  
**개인 일정 소유 + 그룹 공유 + 개인 맞춤 D-day 알림 구조**를 통해  
이 문제를 단순하고 명확하게 해결하는 것을 목표로 합니다.

---

## 2. 핵심 기능 (MVP)

### 🧑 사용자 기능

- 소셜 로그인 (Google / GitHub)
- 그룹 생성
- 그룹 초대 및 참여
- 개인 일정 생성 / 수정 / 삭제
- 그룹 캘린더에서 이름별 일정 조회
- D-day 자동 계산 및 표시
- D-day 기간 동안 매일 알림 발송
- 알림 채널 선택
  - 카카오 알림톡
  - Android 푸시 (FCM)

---

## 3. 시스템 아키텍처

```
Android App (Kotlin)
        ↓
Spring Boot API Server (Java)
        ↓
MySQL Database
        ↓
FCM / Kakao Notification API
```

---

## 4. 기술 스택

### 🖥 Backend
- Java 17
- Spring Boot 3.x
- Spring Security
- OAuth2 Client
- JPA (Hibernate)
- MySQL
- Docker

### 📱 Android App
- Kotlin
- Jetpack Compose
- MVVM Architecture
- Retrofit (REST API 통신)
- Hilt (DI)
- FCM

---

## 5. 데이터베이스 설계 (핵심 테이블)

### User
- id
- email
- name
- provider
- role

### Group
- id
- name
- owner_id

### GroupMember
- group_id
- user_id

### Schedule
- id
- user_id
- title
- schedule_date
- is_important

### ScheduleGroup
- schedule_id
- group_id

### NotificationSetting
- user_id
- dday_enabled
- channel_type

### UserDevice
- user_id
- fcm_token

---

## 6. REST API 설계 (핵심)

인증 방식:  
`Authorization: Bearer <JWT>`

| Endpoint | Method | Description |
|----------|--------|-------------|
| /auth/login/google | POST | Google 로그인 |
| /auth/login/github | POST | GitHub 로그인 |
| /users/me | GET | 현재 사용자 조회 |
| /groups | POST | 그룹 생성 |
| /groups | GET | 내 그룹 목록 |
| /groups/{groupId} | GET | 그룹 상세 |
| /groups/{groupId}/members | GET | 멤버 조회 |
| /schedules | POST | 일정 생성 |
| /schedules | GET | 일정 조회 |
| /schedules/{scheduleId} | PUT | 일정 수정 |
| /schedules/{scheduleId} | DELETE | 일정 삭제 |
| /notifications/settings | GET | 알림 설정 조회 |
| /notifications/settings | PUT | 알림 설정 수정 |
| /devices | POST | FCM 토큰 등록 |
| /internal/trigger-dday | POST | D-day 알림 트리거 (스케줄러용) |

---

## 7. 개발 일정

### Sprint 1
- Spring Boot 초기 세팅 (Java)
- OAuth2 로그인 구현
- 그룹 기능 API 개발
- ERD 확정

### Sprint 2
- 일정 CRUD
- 그룹 일정 공유 로직
- D-day 계산 로직 구현
- FCM 서버 연동

### Sprint 3
- Android 앱 기본 UI (Kotlin + Compose)
- 로그인 연동
- 그룹 / 일정 화면 구현
- 푸시 알림 테스트

---

## 8. 프로젝트 목표

- 그룹 단위 일정 공유의 불편함 해결
- D-day 중심 알림 서비스 구현
- Android 기반 실사용 가능한 MVP 완성
- 확장 가능한 RESTful 구조 설계

---

## 9. 무료 인프라 배포 계획 (MVP)

### 🎯 목표
무료 티어 기반으로 MVP 운영 가능하도록 구성

---

### 인프라 구성

- Backend: Spring Boot (Docker)
- DB: Oracle Cloud Always Free VM + MySQL
- CI/CD: GitHub Actions (SSH 배포)
- Scheduler: GitHub Actions Scheduled Workflow
- Push: FCM (무료)
- Kakao 알림톡: 개발/테스트 가능 (실사용은 유료)

---

### 권장 환경변수

```
DB_HOST
DB_PORT
DB_NAME
DB_USERNAME
DB_PASSWORD
FCM_SERVER_KEY
KAKAO_API_KEY
DEPLOY_TOKEN
VM_USER
VM_HOST
VM_PATH
```

---

### 배포 순서

1. Oracle VM 생성
2. MySQL 설치
3. Docker 설치
4. Spring Boot Dockerize
5. GitHub Actions로 빌드 및 SSH 배포
6. Scheduled workflow로 D-day 알림 트리거

---

⚠ 주의:
- Oracle Always Free 사용 조건 준수
- VM 보안 및 백업 직접 관리 필요
- 카카오 알림톡은 사업자 전환 시 비용 발생

---

© 2026 D-DAY SHARE
