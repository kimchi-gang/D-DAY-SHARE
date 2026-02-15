### 1. users
- provider + provider_id 로 유저 구분
- email은 UNIQUE 안 둠 (GitHub private email 대비)

```sql
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(100),
    name VARCHAR(100),
    provider ENUM('GOOGLE', 'GITHUB') NOT NULL,
    provider_id VARCHAR(100) NOT NULL,
    role ENUM('USER', 'ADMIN') DEFAULT 'USER',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE KEY uk_provider (provider, provider_id)
);
```

<br>

### 2. groups
- invite_code로 초대 링크 구현 가능
- owner는 반드시 존재

```sql
CREATE TABLE groups (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    owner_id BIGINT NOT NULL,
    invite_code VARCHAR(20) UNIQUE,
    invite_expires_at DATETIME NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    CONSTRAINT fk_group_owner
    FOREIGN KEY (owner_id) REFERENCES users(id)
    ON DELETE CASCADE
);

```

<br>

### 3. group_members
- User N : N Group

```sql
CREATE TABLE group_members (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    group_id BIGINT NOT NULL,
    user_id BIGINT NOT NULL,
    role ENUM('OWNER', 'MEMBER') DEFAULT 'MEMBER',
    joined_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    UNIQUE KEY uk_group_user (group_id, user_id),

    CONSTRAINT fk_gm_group
    FOREIGN KEY (group_id) REFERENCES groups(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_gm_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);
```

<br>

### 4. schedules
- 인덱스 확보
- dday_start_offset
        - 7 → 7일 전부터 매일 알림
        - 0 → 당일만 알림

- D-DAY 조회를 빠르게 하기 위해 INDEX 추가

```sql
CREATE TABLE schedules (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    title VARCHAR(150) NOT NULL,
    description TEXT,
    target_date DATE NOT NULL,
    dday_start_offset INT DEFAULT 0,
    is_important BOOLEAN DEFAULT TRUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE,

    INDEX idx_user_active_target (user_id, is_deleted, target_date)

    CONSTRAINT fk_schedule_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);
```

<br>

### 5. schedule_groups (공유용)
- 개인 일정 + 그룹 공유 구조 완성

```sql
CREATE TABLE schedule_groups (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    schedule_id BIGINT NOT NULL,
    group_id BIGINT NOT NULL,

    UNIQUE KEY uk_schedule_group (schedule_id, group_id),

    CONSTRAINT fk_sg_schedule
    FOREIGN KEY (schedule_id) REFERENCES schedules(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_sg_group
    FOREIGN KEY (group_id) REFERENCES groups(id)
    ON DELETE CASCADE
);
```

<br>

### 6. notification_settings
- 1:1 구조
- FCM = Firebase Cloud Messaging

        - 구글이 제공하는 무료 푸시 알림 서비스
```sql
CREATE TABLE notification_settings (
    user_id BIGINT PRIMARY KEY,
    dday_enabled BOOLEAN DEFAULT TRUE,
    channel_type ENUM('FCM', 'KAKAO') DEFAULT 'FCM',

    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT fk_ns_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);
```

<br>

### 7. user_devices (FCM 토큰)
```sql
CREATE TABLE user_devices (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    fcm_token VARCHAR(255) NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    UNIQUE KEY uk_token (fcm_token),
    last_used_at DATETIME NULL,

    CONSTRAINT fk_ud_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE
);
```

<br>

### 8. notifications (발송 로그)
- 운영용으로 로그 (status) 추가

```sql
CREATE TABLE notifications (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    schedule_id BIGINT NOT NULL,
    message TEXT NOT NULL,
    channel ENUM('FCM', 'KAKAO') NOT NULL,
    status ENUM('SUCCESS', 'FAIL') DEFAULT 'SUCCESS',
    sent_at DATETIME DEFAULT CURRENT_TIMESTAMP,

    error_message VARCHAR(255) NULL,

    CONSTRAINT fk_not_user
    FOREIGN KEY (user_id) REFERENCES users(id)
    ON DELETE CASCADE,

    CONSTRAINT fk_not_schedule
    FOREIGN KEY (schedule_id) REFERENCES schedules(id)
    ON DELETE CASCADE
);
```

<br>

### 9. schedule_notifications (일정별 알림 설정)

```sql
CREATE TABLE schedule_notifications (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    schedule_id BIGINT NOT NULL,

    notify_hour INT NOT NULL,
    notify_minute INT NOT NULL,

    repeat_type ENUM('DAILY', 'WEEKDAY', 'CUSTOM') DEFAULT 'DAILY',
    repeat_days INT DEFAULT NULL,

    enabled BOOLEAN DEFAULT TRUE,
    timezone VARCHAR(50) DEFAULT 'Asia/Seoul',

    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP 
        ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_sn_schedule_enabled (schedule_id, enabled),

    CONSTRAINT fk_sn_schedule
    FOREIGN KEY (schedule_id) REFERENCES schedules(id)
    ON DELETE CASCADE
);
```