# Week 2 - Rocky Linux 기본 명령어

## 사용자 및 그룹 관리

### 주요 개념
- 리눅스는 다중 사용자 시스템으로, 모든 사용자는 하나 이상의 그룹에 소속
- 사용자 정보: `/etc/passwd`
- 비밀번호 정보: `/etc/shadow`
- 그룹 정보: `/etc/group`

### 사용자 관련 명령어
```bash
useradd newuser              # 사용자 생성
useradd -g mygroup newuser   # 그룹 지정하여 사용자 생성
passwd newuser               # 비밀번호 설정
usermod -g root newuser      # 사용자 그룹 변경
userdel -r newuser           # 사용자 삭제 (홈 디렉터리 포함)
```

### 그룹 관련 명령어
```bash
groupadd mygroup             # 그룹 생성
groupmod -n newname mygroup  # 그룹명 변경
groupdel mygroup             # 그룹 삭제
gpasswd -a user1 mygroup     # 그룹에 사용자 추가
```

**Best Practice**: 먼저 그룹을 만들고 사용자를 생성하여 소속시키기

---

## 파일 권한 및 소유권

### 권한 구조
```
-rwxr-xr--
```
- 파일 유형: `-` (파일), `d` (디렉터리), `l` (심볼릭 링크)
- 소유자(User) / 그룹(Group) / 기타(Other) 순서로 3개씩
- r(읽기=4), w(쓰기=2), x(실행=1)

### chmod - 권한 변경
```bash
# 숫자 방식
chmod 755 file.txt           # rwxr-xr-x
chmod 644 file.txt           # rw-r--r--

# 문자 방식
chmod u+x file.txt           # 소유자에게 실행 권한 추가
chmod g-w file.txt           # 그룹 쓰기 권한 제거
chmod o+r file.txt           # 기타 사용자 읽기 권한 추가
```

### chown/chgrp - 소유권 변경
```bash
chown rocky file.txt         # 소유자 변경
chown rocky:rocky file.txt   # 소유자와 그룹 동시 변경
chgrp rocky file.txt         # 그룹 변경
```

**참고**: 소유권 변경은 root 사용자만 가능

---

## 링크

### 하드 링크 vs 심볼릭 링크
| 구분 | 하드 링크 | 심볼릭 링크 |
|------|-----------|-------------|
| inode | 원본과 동일 | 새로 생성 |
| 원본 삭제시 | 영향 없음 | 연결 끊김 |
| 생성 명령 | `ln basefile hardlink` | `ln -s basefile softlink` |

```bash
ln basefile hardlink         # 하드 링크 생성
ln -s basefile softlink      # 심볼릭 링크 생성
ls -il                       # inode 번호 확인
```

---

## 패키지 관리

### DNF (Dandified YUM)
RPM의 의존성 문제를 해결하는 패키지 관리 도구

```bash
# 기본 명령어
dnf install 패키지명          # 패키지 설치
dnf remove 패키지명           # 패키지 제거
dnf update 패키지명           # 패키지 업데이트
dnf info 패키지명             # 패키지 정보 확인
dnf list httpd*              # 패키지 목록 검색

# 고급 명령어
dnf groupinstall "그룹명"     # 패키지 그룹 설치
dnf check-update             # 업데이트 가능 목록 확인
dnf clean all                # 캐시 정리
```

### DNF 작동 원리
1. `/etc/yum.repos.d/` 디렉터리의 repo 파일에서 URL 확인
2. 저장소에서 메타데이터 다운로드
3. 의존성 계산 후 설치 목록 확정
4. 패키지 다운로드 및 설치

---

## 파일 압축 및 묶기

### 압축 명령어
```bash
# xz 압축 (최신, 높은 압축률)
xz file.txt                  # 압축
xz -d file.txt.xz            # 압축 해제
xz -k file.txt               # 원본 파일 유지하며 압축

# zip
zip test.zip file.txt        # 압축 (원본 유지)
unzip test.zip               # 압축 해제
```

### tar - 파일 묶기
```bash
# 묶기
tar cvf archive.tar /path/   # tar로 묶기
tar cvfJ archive.tar.xz /path/  # tar + xz 압축

# 확인 및 풀기
tar tvf archive.tar          # 내용 확인
tar xvf archive.tar          # 풀기
tar xvfJ archive.tar.xz      # xz 압축 해제 + 풀기
tar xvf archive.tar -C /newdir  # 특정 디렉터리에 풀기
```

**tar 옵션**
- `c`: 생성, `x`: 해제, `t`: 내용 확인
- `f`: 파일명 지정 (필수)
- `v`: 진행 과정 표시
- `J`: xz, `z`: gzip, `j`: bzip2

---

## 파일 검색

```bash
find /etc -name "*.conf"     # 패턴으로 파일 검색
which ls                     # 실행 파일 경로 찾기
whereis ls                   # 실행 파일 및 관련 파일 위치
locate file.txt              # 파일 데이터베이스에서 검색
```

---

## 시스템 설정 도구

```bash
nmtui                        # 네트워크 설정
firewall-config              # 방화벽 설정
ntsysv                       # 서비스(데몬) 설정
```

---

## 학습 정리

### inode란?
- 파일/디렉터리의 메타데이터를 저장하는 데이터 구조
- 파일의 허가권, 소유권, 데이터 블록 위치 등 포함
- 모든 파일은 고유한 inode 번호를 가짐
- 확인: `ls -li` (inode 번호), `stat` (전체 정보)

### 리포지토리 종류
- **baseos**: 핵심 시스템 구성요소
- **appstream**: 사용자 소프트웨어, 프로그래밍 언어
- **extras**: 추가 패키지
- **crb**: 개발 도구 및 라이브러리
