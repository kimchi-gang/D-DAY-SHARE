# D-DAY-SHARE
“일정을 공유하고, 나에게 중요한 날의 D-day를 매일 카카오톡으로 알림 받자”


# 📘 프로젝트 개발 문서
---

## 1. 프로젝트 개요

### 📌 프로젝트 명
**D-DAY SHARE**

### 📖 프로젝트 설명

D-DAY SHARE는 스터디, 팀, 가족 등 **그룹 단위 사용자**가  
각자의 중요한 일정을 이름별로 공유하고,  
개인 설정에 따라 D-day 알림을 받을 수 있는 서비스입니다.

기존 캘린더 서비스는 일정 공유는 가능하지만  
- 누가 만든 일정인지 구분이 어렵고  
- 알림이 개인화되어 있지 않으며  
- D-day 중심 관리가 불편하다는 문제가 있습니다.

본 서비스는  
**개인 일정 소유 + 그룹 공유 + 개인 맞춤 D-day 알림 구조**를 통해  
이 문제를 단순하고 명확하게 해결하는 것을 목표로 합니다.

---

## 2. 핵심 기능 (최소 기능 목록 - MVP)

### 🧑 사용자 기능ㅉ

- 소셜 로그인 (Google / GitHub)
- 그룹 생성
- 그룹 초대 및 참여
- 개인 일정 생성 / 수정 / 삭제
- 그룹 캘린더에서 이름별 일정 조회
- D-day 자동 계산 및 표시
- D-day 기간 동안 매일 알림 발송
- 알림 채널 선택 (카카오톡 / 모바일 푸시 / 이메일)

---

### 🔔 알림 정책 (초기 버전)

- D-day 기준 자동 계산
- 중요한 날 전까지 매일 알림
- 사용자별 알림 ON/OFF 가능
- 알림 채널 선택 가능
  - 카카오 알림톡
  - Android 푸시 (FCM)

---

## 3. MVP 범위

### 🚀 목표

그룹 기반 일정 공유 + D-day 알림 기능이 완전히 동작하는 최소 제품 구현

### ✅ 포함 기능

- OAuth2 로그인
- 그룹 생성 / 초대 / 참여
- 일정 CRUD
- 그룹 공유 기능
- D-day 계산 로직
- Android 푸시 알림
- 카카오 알림톡 연동

### ❌ 제외 기능 (v2 이후)

- 반복 일정
- 그룹 공용 일정
- 채팅 기능
- 통계 기능
- 웹 푸시

---

## 4. 시스템 아키텍처

```
Client (Web / Android)
        ↓
Spring Boot API Server
        ↓
MySQL Database
        ↓
Firebase Cloud Messaging
        ↓
Kakao Notification API
```

---

## 5. 기술 스택

### Backend
- Spring Boot 3.x
- Spring Security
- OAuth2
- JWT
- MySQL 8.0
- Docker

### Frontend
- React
- TypeScript
- React Native (Android)

### Notification
- Firebase Cloud Messaging (FCM)
- Kakao 알림톡 API

---

## 6. 데이터베이스 설계 (핵심 테이블)

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

## 7. API 설계 (핵심)

| Endpoint | Method | Description |
|----------|--------|-------------|
| /auth/login/google | POST | Google 로그인 (OAuth callback/token 발급) |
| /auth/login/github | POST | GitHub 로그인 |
| /auth/logout | POST | 로그아웃 / 토큰 무효화 |
| /users/me | GET | 현재 사용자 정보 조회 |
| /groups | POST | 그룹 생성 |
| /groups | GET | 사용자가 속한 그룹 목록 조회 |
| /groups/{groupId} | GET | 그룹 상세 조회 |
| /groups/{groupId} | PUT | 그룹 정보 수정 |
| /groups/{groupId} | DELETE | 그룹 삭제 (owner 권한) |
| /groups/{groupId}/invite | POST | 초대 링크/초대 메일 발송 |
| /groups/{groupId}/members | GET | 그룹 멤버 목록 조회 |
| /groups/{groupId}/members | POST | 그룹에 사용자 초대/추가 |
| /groups/{groupId}/members/{userId} | DELETE | 그룹에서 사용자 제거 |
| /schedules | POST | 일정 생성 (개인 일정) |
| /schedules | GET | 사용자(또는 그룹) 일정 목록 조회 (쿼리: groupId, from, to) |
| /schedules/{scheduleId} | GET | 일정 상세 조회 |
| /schedules/{scheduleId} | PUT | 일정 수정 |
| /schedules/{scheduleId} | DELETE | 일정 삭제 |
| /schedules/{scheduleId}/share | POST | 일정 그룹 공유(연결) |
| /notifications/settings | GET | 현재 사용자 알림 설정 조회 |
| /notifications/settings | PUT | 알림 설정 업데이트 (채널, D-day ON/OFF 등) |
| /notifications/send | POST | (관리용) 특정 사용자/그룹에 알림 즉시 전송 (관리자 또는 내부용) |
| /devices | POST | 기기 등록(FCM 토큰 등록) |
| /devices | GET | 등록된 기기 목록 조회 |
| /devices/{deviceId} | DELETE | 기기 등록 해제 |
| /internal/trigger-dday | POST | (시스템 내부) 스케줄러에서 호출 — 오늘 D-day 대상에 대해 알림 생성/발송 트리거 |


Notes: 인증이 필요한 엔드포인트는 `Authorization: Bearer <JWT>` 형태 사용. 입력 검증, 페이로드 스펙은 API 문서에 상세 정의 예정.

---

## 8. 개발 일정

### Sprint 1
- 프로젝트 초기 세팅
- 로그인 구현
- 그룹 기능 구현
- ERD 확정

### Sprint 2
- 일정 CRUD
- 그룹 공유 기능
- D-day 계산 로직
- 푸시 알림 연동

---

## 9. 프로젝트 목표

- 그룹 단위 일정 공유의 불편함 해결
- D-day 중심 알림 서비스 구현
- Android 기반 실사용 가능한 MVP 완성
- 추후 확장 가능한 구조 설계

---

## 10. 향후 확장 방향

- 반복 일정 기능
- 그룹 공용 일정
- 웹 버전 확장
- 알림 세부 설정 (D-7, D-3, D-1 선택형)
- 통계 및 히스토리 기능

---

## 11. 무료 인프라 배포 계획 (MVP)

목표: 비용을 최소화하여 MVP를 운영 가능하게 만드는 배포 전략과 구체적 작업 항목을 정리합니다. 전체 스택은 무료 티어 위주로 구성하되, 실사용(대량 메시지 발송 등)은 별도 비용이 필요함을 명시합니다.
목표: 무료 티어로 MVP를 운영할 수 있도록 최소 구성만 명시합니다.


환경변수(권장): `JDBC_DATABASE_URL`, `DB_USERNAME`, `DB_PASSWORD`, `FCM_SERVER_KEY`, `KAKAO_API_KEY`, `DEPLOY_TOKEN`

핵심 순서(간단): DB 생성 → 환경변수로 연결 설정 → Dockerize → GitHub Actions로 빌드·배포 → Scheduled workflow로 알림 트리거

주의: 무료 티어의 성능/동시성 제약과 Kakao의 유료 전환 필요성을 고려하세요.
목표: 항상 무료로 운영 가능한 DB(Oracle Cloud Always Free VM) 기준의 간단 배포 계획입니다.

- Backend: Spring Boot (Docker) — VM에 JAR 배포 또는 Docker 실행
- Frontend: React → Vercel / Netlify
- DB: Oracle Cloud Always Free VM에 설치한 MySQL (항상 무료 제공 조건에 따름)
- CI/CD: GitHub Actions (빌드 · SSH로 VM에 배포)
- Scheduler: GitHub Actions 스케줄러로 매일 알림 트리거
- Push: FCM (Android 무료)
- Kakao 알림톡: 개발/테스트 가능 — 실사용은 사업자 계정(유료)

환경변수(권장): `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`, `FCM_SERVER_KEY`, `KAKAO_API_KEY`, `DEPLOY_TOKEN`, `VM_USER`, `VM_HOST`, `VM_PATH`

핵심 순서(간단): Oracle VM 생성 → MySQL 설치 → 환경변수 설정 → Dockerize 또는 JAR 복사 → GitHub Actions로 빌드·SSH 배포 → Scheduled workflow로 알림 트리거

주의: Oracle의 Always Free 조건(소비량·리전 제한 등)을 준수해야 하며, VM 운영(보안·백업)은 직접 관리해야 합니다.
---

© 2026 D-DAY SHARE