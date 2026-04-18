# linux-study

리눅스(Ubuntu) 환경에서 직접 서버를 구성하고,  
사용자/권한/네트워크 문제를 해결하며 학습한 내용을 기록한 저장소입니다.

단순 명령어 정리가 아니라 실제 서비스 동작과 트러블슈팅 경험을 중심으로 구성했습니다.

---

## 실습 환경 아키텍처

[Windows 10] ──SSH(PuTTY)──▶ [Ubuntu 24.04 / VMware]
     │                              │
     └──FTP(21)──────────▶  Apache2(80) / vsftpd
                                   │
                            /var/www/html/index.html

---

## 1. 학습 개요
- 환경: Ubuntu 24.04 (VMware), PuTTY(SSH)
- 범위: 사용자 관리, 권한, 웹/FTP 서버 구축
- 목표: 단순 명령어 암기가 아닌 실제 서버 운영 경험 확보

---

## 2. 트러블슈팅

### 🔹 403 Forbidden 발생
- 문제: 웹 페이지 접근 불가
- 원인: index.html 권한 600
- 해결: chmod 644 index.html
- 결과: 정상 출력

웹 서버는 모든 사용자가 접근해야 하므로 other 읽기 권한이 필요하다는 점을 확인했다.

---

### 🔹 FTP 업로드 실패
- 문제: Permission denied
- 원인: vsftpd write_enable 비활성화
- 해결: 설정 변경 후 재시작
- 결과: 업로드 성공

서비스 설정 문제와 파일 권한 문제를 구분하는 것이 중요하다는 점을 이해했다.

---

### 🔹 ping 통신 불가
- 문제: 응답 없음
- 원인: Windows 방화벽 ICMP 차단
- 해결: ICMP 허용 설정
- 결과: 통신 정상

네트워크 문제와 서버 설정 문제를 구분하는 기준을 익혔다.

---

## 3. 실습 결과

- Apache 웹 서버 구축 및 페이지 배포 완료
- FTP 서버를 통한 외부 파일 업로드 성공
- 권한 변경에 따른 서비스 동작 변화 확인

---

## 4. 핵심 개념 정리

- chmod 755 → 디렉터리 접근 가능
- chmod 644 → 웹 서비스 필수 권한
- 디렉터리는 x 권한이 없으면 접근 자체 불가
- 권한 문제는 서비스 장애로 직결됨
- 문제 발생 시 권한 / 설정 / 네트워크 기준으로 원인 분석

---

## 5. 명령어 정리

### 📂 파일 및 디렉터리

| 명령어 | 설명 | 예제 |
|--------|------|------|
| pwd | 현재 위치 확인 | pwd |
| ls | 목록 확인 | ls -l |
| cd | 디렉터리 이동 | cd /home |
| mkdir | 디렉터리 생성 | mkdir test |
| rm | 삭제 | rm -r test |
| cp | 파일 복사 | cp a.txt b.txt |
| mv | 이동/이름 변경 | mv a.txt b.txt |
| touch | 빈 파일 생성 | touch file.txt |

---

### 📄 파일 내용 확인

| 명령어 | 설명 | 예제 |
|--------|------|------|
| cat | 전체 출력 | cat file.txt |
| head | 상단 출력 | head file.txt |
| tail | 하단 출력 | tail file.txt |
| grep | 문자열 검색 | grep test file.txt |

---

### 👤 사용자 관리

| 명령어 | 설명 | 예제 |
|--------|------|------|
| useradd | 사용자 생성 | useradd -m user1 |
| usermod | 사용자 수정 | usermod -d /home/user1 user1 |
| userdel | 사용자 삭제 | userdel -r user1 |
| passwd | 비밀번호 설정 | passwd user1 |

---

### 🔐 권한 및 소유권

| 명령어 | 설명 | 예제 |
|--------|------|------|
| chmod | 권한 변경 | chmod 755 file |
| chown | 소유자 변경 | chown user:group file |

---

### 📝 vi 에디터

| 명령어 | 설명 |
|--------|------|
| i | 입력 모드 |
| esc | 명령 모드 |
| :w | 저장 |
| :q | 종료 |
| :wq | 저장 후 종료 |
| :q! | 저장 없이 강제 종료 |

---

### 💡 실습에서 실제 사용한 명령어

```bash
# 웹 서버 권한 문제 해결
chmod 644 /var/www/html/index.html

# FTP 설정 변경 및 재시작
vi /etc/vsftpd.conf
systemctl restart vsftpd

# 사용자 생성 및 환경 설정
useradd -m -d /home/user1 -s /bin/bash user1

# 로그 및 파일 내용 검색
grep "error" /var/log/syslog
