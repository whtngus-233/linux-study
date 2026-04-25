# linux-study

> Ubuntu 서버에서 직접 겪은 **403 Forbidden / Permission denied / ping 차단**  
> 트러블슈팅 3건을 권한·서비스·네트워크 레이어로 분리 분석한 학습 기록

리눅스(Ubuntu) 환경에서 직접 서버를 구성하고  
사용자 / 권한 / 네트워크 문제를 해결하며 학습한 내용을 기록한 저장소입니다.

단순 명령어 정리가 아니라 **실제 서비스 동작과 트러블슈팅 경험**을 중심으로 구성했습니다.

---

## 🎯 이 레포에서 볼 수 있는 것

- 같은 에러(403)도 원인은 **파일 권한 / 디렉터리 권한 / 서비스 설정**으로 나뉜다는 레이어별 분석
- `chmod 777`로 끝내지 않고 **vsftpd 설정**까지 파고든 디버깅 과정
- **클라이언트 → FTP → Apache** 전체 서비스 플로우를 직접 구성한 실습 기록
- 트러블슈팅 케이스별 **블로그 상세 분석 글**과 연결된 포트폴리오 구조

---

## 📖 트러블슈팅 시리즈 (블로그 상세 분석)

이 저장소의 트러블슈팅 케이스는 블로그에 시리즈로 상세 분석되어 있습니다.

- [1편: Apache 403 — 파일 권한 문제](https://velog.io/@whtngus233/Linux-%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85-%EC%8B%A4%EC%A0%84-%EB%B6%84%EC%84%9D-1%ED%8E%B8-Apache-403-%ED%8C%8C%EC%9D%BC-%EA%B6%8C%ED%95%9C-%EB%AC%B8%EC%A0%9C)
- [2편: Apache 403 — 디렉터리 x 권한 문제](https://velog.io/@whtngus233/%ED%8C%8C%EC%9D%BC-%EA%B6%8C%ED%95%9C%EC%9D%B4-%EC%A0%95%EC%83%81%EC%9D%B4%EB%8D%94%EB%9D%BC%EB%8F%84-%EA%B2%BD%EB%A1%9C-%EC%A0%91%EA%B7%BC-%EA%B6%8C%ED%95%9C%EC%9D%B4-%EC%97%86%EC%9C%BC%EB%A9%B4-403%EC%9D%B4-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)
- [3편: vsftpd 업로드 실패 — write_enable 설정](https://velog.io/@whtngus233/Permission-denied%EC%9D%98-%EC%9B%90%EC%9D%B8%EC%9D%B4-chmod%EA%B0%80-%EC%95%84%EB%8B%88%EB%9D%BC-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%84%A4%EC%A0%95%EC%9D%B4%EC%97%88%EB%8D%98-%EC%82%AC%EB%A1%80-%EB%B6%84%EC%84%9D)

---

## 🧱 실습 환경 아키텍처

```
[ Host: Windows 11 ] 
      │
      │ (1) SSH 접속 (PuTTY / 192.168.10.128)
      ▼
┌────────────────── [ VMware Workstation ] ──────────────────┐
│                                                            │
│  [ Guest 1: Windows 11 ]        [ Guest 2: Ubuntu 24.04 ]  │
│  (Client Role)                  (Server Role: .128)        │
│          │                                 │               │
│          │ (2) FTP 접속 & 파일 전송         │              │
│          └────────────────────────────────▶│── [vsftpd]   │
│            (index.html 업로드)              │      │       │
│                                            │ (3) mv 명령   │
│                                            │      ▼        │
│                                            └── [Apache2]   │
│                                                    │       │
└────────────────────────────────────────────────────┼───────┘
      ▲                                              │
      └─────── (4) 웹 브라우저 접속 (http://192.168.10.128) ──┘
```

---

## 🔄 실습 흐름

본 실습은 클라이언트 → 서버 → 웹 서비스 구조로 진행되었습니다.

1. **Windows 11 (클라이언트)**
   - PuTTY를 통해 Ubuntu 서버에 SSH 접속
   - CMD에서 FTP 접속을 통해 파일 업로드 수행

2. **Ubuntu 24.04 (서버)**
   - FTP(vsftpd)를 통해 클라이언트 파일 수신
   - 업로드된 파일을 웹 서버 경로(`/var/www/html`)로 이동

3. **Apache 웹 서버**
   - `index.html` 파일을 읽어 브라우저에 제공
   - 웹 페이지 정상 출력 확인

---

## 📸 실습 스크린샷

### 1. 403 Forbidden 발생
브라우저에서 `http://192.168.10.128` 접속 시 403 응답 —  
원인은 `index.html` 권한이 `600`이어서 Apache가 읽을 수 없었기 때문.

<img width="873" height="689" alt="image" src="https://github.com/user-attachments/assets/78295ca5-d61c-4f4e-92cc-d584c7ac305a" />

### 2. 정상 출력
파일 권한을 `644`로 변경한 뒤 브라우저에서 정상적으로 페이지가 출력되는 모습.

<img width="876" height="435" alt="image" src="https://github.com/user-attachments/assets/b9c52e34-30f4-47dc-9eb9-d0e505aca02e" />

### 3. chmod 변경 전/후
`chmod 644` 적용 전(`-rw-------`)과 적용 후(`-rw-r--r--`) 권한 비교.

<img width="741" height="232" alt="image" src="https://github.com/user-attachments/assets/c7eb2274-8d81-4443-bd42-bd7430ae0cec" />

### 4. FTP 업로드 성공
`vsftpd.conf`의 `write_enable=YES` 활성화 후 클라이언트에서 파일 업로드 성공.

<img width="876" height="688" alt="image" src="https://github.com/user-attachments/assets/baa8bdbc-c9e1-4352-afc0-84097d8462ed" />

---

## 1. 학습 개요

- 환경: Ubuntu 24.04 (VMware), PuTTY(SSH)
- 범위: 사용자 관리, 권한, 웹/FTP 서버 구축
- 목표: 단순 명령어 암기가 아닌 실제 서버 운영 경험 확보

---

## 2. 트러블슈팅

### 🔹 403 Forbidden 발생

- 문제: 웹 페이지 접근 불가  
- 원인: `index.html` 권한 `600`  
- 해결: `chmod 644 index.html`  
- 결과: 정상 출력  

웹 서버는 파일을 실행하는 것이 아니라 읽어서 제공하므로  
**other 사용자에게 읽기 권한이 반드시 필요하다.**

📖 상세 분석: [1편 — Apache 403: 파일 권한 문제](https://velog.io/@whtngus233/Linux-%ED%8A%B8%EB%9F%AC%EB%B8%94%EC%8A%88%ED%8C%85-%EC%8B%A4%EC%A0%84-%EB%B6%84%EC%84%9D-1%ED%8E%B8-Apache-403-%ED%8C%8C%EC%9D%BC-%EA%B6%8C%ED%95%9C-%EB%AC%B8%EC%A0%9C)

---

### 🔹 디렉터리 x 권한 누락으로 403 발생

- 문제: 파일 권한이 `644`인데도 403 발생  
- 원인: `/var/www/html` 디렉터리에 `x` 권한 없음  
- 해결: `chmod 755 /var/www/html`  
- 결과: 정상 출력  

디렉터리에서 `x`는 실행이 아니라 **경로 통과 권한**이며,  
파일 권한이 정상이어도 상위 디렉터리에 `x`가 없으면 접근 자체가 차단된다.

📖 상세 분석: [2편 — Apache 403: 디렉터리 x 권한 문제](https://velog.io/@whtngus233/%ED%8C%8C%EC%9D%BC-%EA%B6%8C%ED%95%9C%EC%9D%B4-%EC%A0%95%EC%83%81%EC%9D%B4%EB%8D%94%EB%9D%BC%EB%8F%84-%EA%B2%BD%EB%A1%9C-%EC%A0%91%EA%B7%BC-%EA%B6%8C%ED%95%9C%EC%9D%B4-%EC%97%86%EC%9C%BC%EB%A9%B4-403%EC%9D%B4-%EB%B0%9C%EC%83%9D%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)

---

### 🔹 FTP 업로드 실패

- 문제: `Permission denied`  
- 원인: `vsftpd`의 `write_enable` 비활성화  
- 해결: 설정 변경 후 서비스 재시작  
- 결과: 업로드 성공  

FTP 접속 성공과 파일 업로드 가능 여부는 별개이며  
**서비스 설정과 파일 권한은 구분해서 확인해야 한다.**

📖 상세 분석: [3편 — vsftpd 업로드 실패와 write_enable 설정](https://velog.io/@whtngus233/Permission-denied%EC%9D%98-%EC%9B%90%EC%9D%B8%EC%9D%B4-chmod%EA%B0%80-%EC%95%84%EB%8B%88%EB%9D%BC-%EC%84%9C%EB%B9%84%EC%8A%A4-%EC%84%A4%EC%A0%95%EC%9D%B4%EC%97%88%EB%8D%98-%EC%82%AC%EB%A1%80-%EB%B6%84%EC%84%9D)

---

### 🔹 ping 통신 불가

- 문제: 응답 없음  
- 원인: Windows 방화벽 ICMP 차단  
- 해결: ICMP 허용 설정  
- 결과: 통신 정상  

네트워크 문제는 단순 연결 문제가 아니라  
**방화벽 정책까지 포함해서 판단해야 한다.**

---

## 3. 실습 결과

- Apache 웹 서버 구축 및 페이지 배포 완료
- FTP 서버를 통한 외부 파일 업로드 성공
- 권한 변경에 따른 서비스 동작 변화 확인

---

## 4. 핵심 개념 정리

- `chmod 755` → 디렉터리 접근 가능 (`x` 권한 필요)
- `chmod 644` → 웹 서비스 파일 기본 권한
- 디렉터리는 `x` 권한이 없으면 접근 자체 불가
- 웹 서버는 파일을 "실행"하는 것이 아니라 **"읽어서 제공"** 한다
- 문제 발생 시 **권한 / 설정 / 네트워크 기준으로 분리해서 분석**

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

### 📝 vi 에디터 명령어

#### 모드 전환
| 명령 | 설명 |
|------|------|
| i | 입력 모드 |
| esc | 명령 모드 |
| :w | 저장 |
| :q | 종료 |
| :wq | 저장 후 종료 |

#### 이동
| 명령 | 설명 |
|------|------|
| h j k l | 방향 이동 |
| gg / G | 맨 위 / 맨 아래 |
| 0 / $ | 줄 시작 / 끝 |

#### 편집
| 명령 | 설명 |
|------|------|
| dd | 한 줄 삭제 |
| yy | 한 줄 복사 |
| p | 붙여넣기 |
| u | 실행 취소 |

---

## 6. 실습에서 사용한 명령어

```bash
# 웹 서버 권한 문제 해결
chmod 644 /var/www/html/index.html

# FTP 설정 변경
vi /etc/vsftpd.conf
systemctl restart vsftpd

# 사용자 생성
useradd -m -d /home/user1 -s /bin/bash user1

# 로그 검색
grep "error" /var/log/syslog
```

---

## 7. 커밋 규칙

- `docs`: 문서 작성
- `feat`: 기능/실습 추가
- `fix`: 오류 수정
- `chore`: 이미지 및 기타 작업
- `refactor`: 구조 개선

---

## 📌 정리

이 저장소는 단순한 리눅스 명령어 정리가 아니라  
**클라이언트 → 서버 → 웹 서비스 흐름을 직접 구성하고 문제를 해결한 경험을 정리한 포트폴리오입니다.**
