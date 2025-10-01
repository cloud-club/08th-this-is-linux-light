## 사용자와 그룹 관련 명령

### Command
- 사용자 계정의 기본 정보
`gedit /etc/passwd`

### Result
```bash
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...
rocky:x:1000:1000:rocky:/home/rocky:/bin/bash
```

### Lesson
- 리눅스는 다중 사용자 시스템. 1대의 리눅스에 여러 명의 사용자가 동시에 접속 가능
	- 모든 사용자는 하나 이상의 그룹에 소속되어야 한다
	- 사용자는 여러 그룹에 속할 수 있다
		- 주요 그룹 1개와 보조 그룹 여러개
- 파일 구조 `username:password:UID:GID:GECOS:home_directory:shell`
	- username - 사용자 이름
	- passwd - 패스워드 (실제 패스워드는 `/etc/shadow`에 암호화되어 저장)
	- UID - 사용자 ID
	- GID - 그룹 ID
	- GECOS - 전체 이름
	- home_directory - 홈 디렉토리 경로
	- shell - 기본 쉘 프로그램

---

### Command
- 시스템에 정의된 모든 그룹 정보
`gedit /etc/group`

### Result
```bash
root:x:0:
bin:x:1:
...
rocky:x:1000:
```

### Lesson
- 파일 구조 `group_name:password:GID:user_list`
	- group_name - 그룹 이름
	- password - 그룹 패스워드 (실제 패스워드는 `/etc/gshadow`)
	- GID - 그룹 ID
	- user_list - 이 그룹에 속한 사용자들의 목록
- user_list는 참조로 사용된다. 아무것도 써 있지 않다고 그룹에 소속된 사용자가 없다는 뜻은 아니다

---

### Command
- 사용자관련 
```bash
# useradd
useradd newuser # newuser라는 사용자 생성
useradd -u 1111 newuser # newuser 사용자를 생성 및 UID를 1111로 지정
useradd -g mygroup newuser # newuser 사용자 생성 및 mygroup에 추가
useradd -g main -G sub user1 # user1 사용자 생성 및 main을 주요 그룹으로, sub을 보조 그룹으로 가입시킴
useradd -d /newhome newuser # newuser 사용자 생성 및 홈 디렉터리 /newhome 지정
useradd -s /bin/csh newuser # newuser 사용자 생성 및 기본 셸 /bin/csh 지정

# passwd
passwd newuser # newuser 사용자의 비밀번호 지정 또는 변경

# usermod
usermod -g root newuser # newuser 사용자의 그룹을 root 그룹으로 변경

# userdel
userdel newuser # newuser 삭제
userdel -r newuser # newuser 삭제 및 홈디렉터리도 함께 삭제

# chage
chage -l newuser # newuser 사용자에 설정된 사항 확인
chage -m 2 newuser # 비밀번호 변경 후 최소 2일은 사용하도록
chage -M 30 newuser # 비밀번호 변경 후 최대 30일까지 사용할 수 있음
chage -E 2030/12/12 newuser # 비밀번호 만료일 설정
chage -W 10 newuser # 비밀번호 만료 10일 전부터 경고
```

- 그룹관련
```bash
# groups
groups # 현재 사용자가 소속된 그룹 표시
groups newuser # newuser 사용자가 소속된 그룹 표시

# groupadd
groupadd mygroup # mygroup 생성
groupadd -g 2222 mygroup #  그룹 생성 및 GID 2222 지정

# groupmod
groupmod -n mymy mygroup # mymy로 그룹명 변경

# groupdel
groupdel mygroup # mygroup 삭제

# gpasswd
gpasswd newgroup # newgroup 비밀번호 지정
gpasswd -A newuser newgroup # newuser 사용자를 newgroup 의 관리자로 지정
gpasswd -a user1 newgroup # user1을 newgroup 의 사용자로 추가
gpasswd -d newuser newgroup # user1을 newgroup 의 사용자에서 제거
```

### Lesson
- `groupdel`명령어로 그룹 삭제시 해당 그룹을 주요 그룹으로 지정한 사용자가 없어야 함

## 사용자와 그룹 관련 명령 - 실습

### Command
- 사용자 , 그룹 관리
```bash
groupadd rockyGroup
tail -5 /etc/group # 그룹 확인

useradd -g rockyGroup user1
useradd -g rockyGroup user2
tail -5 /etc/passwd # 그룹 ID 확인

ls -a /home/user1
ls -a /etc/skel

userdel -r user1
userdel -r user2
groupdel rockyGroup
```

### Lesson
- `useradd` 명령을 실행해 별도의 그룹을 지정하지 않으면 자동으로 사용자 이름과 동일한 그룹이 생성되며 해당 그룹에 자동으로 포함된다
	- 따라서 **먼저 그룹을 만들고 사용자를 속하도록 생성하는 것**이 best practice
- `/etc/shadow`의 비밀번호 부분 `!!`은 사용자의 비밀번호가 지정되지 않음을 의미
- root 사용자로 다른 사용자의 비밀번호를 지정할 때는 간단한 비밀번호도 사용 가능
- `/etc/skel` 디렉터리의 모든 내용이 사용자의 홈 디렉터리에 복사된다
	- 생성할 사용자에게 특정 파일을 배포하고 싶다면 `/etc/skel` 디렉터리에 넣어 두면 된다(skel은 skeleton의 약자)

## 파일, 디렉터리의 소유와 허가권

### Command
- 파일의 소유,허가 정보 확인
```bash
touch test.txt
ls -l
```

### Result
```bash
-rw-r--r--  1 root root    0  9월 28 21:33 test.txt
...
```

### Lesson
- 각 파일과 디렉터리마다 소유권과 허가권이라는 속성이 있다
- `-rw-r--r--  1 root root    0  9월 28 21:33 test.txt`
	- 파일 유형 (가장 맨 앞) `-`
		- `-` - 일반 파일
		- `d` - 디렉터리
		- `l` - 심볼릭 링크
		- `c` - 캐릭터 디바이스
		- `b` - 블록 디바이스
	- 파일 허가권 `rw-r--r--` 9개 문자
		- 권한별 의미
			- r - read 읽기 권한, 4
			- w - write 쓰기 권한, 2
			- x - execute 실행 권한, 1
				- `rw-r--r-x`를 숫자로 나타내면 645
		- 3개씩 나누어 소유자(User), 그룹(Group), 기타(Other) 3그룹으로 나뉨
	- 링크 수 `1`
	- 파일 소유자 이름 `root`
	- 파일이 속한 그룹 `root`
	- 파일 크기 `0`
	- 마지막 수정 시간
	- 파일명

---

### Command
- `chmod` - 파일 허가권을 변경하는 명령
```bash
touch test.txt # -rw-r--r--

# 숫자방식
chmod 777 test.txt # -rwxrwxrwx, 모든 권한
chmod 744 test.txt # -rwxr--r--, 소유자만 모든 권한 나머지 읽기만
chmod 640 test.txt # -rw-r-----, 소유자 읽기+쓰기, 그룹 읽기, 기타 권한없음

chmod 644 test.txt

# 문자방식
chmod u+x test.txt # -rwxr--r--, 소유자에게 실행 권한 추가
chmod g-r test.txt # -rwx---r--, 그룹에서 읽기 권한 제거
chmod o+w test.txt # -rwx---r-x, 기타 사용자에게 쓰기 권한 추가
chmod g=rw test.txt # -rwxrw-r-x, 그룹 권한을 읽기+쓰기로 설정

# 여러 권한 한 번에 설정
chmod u=rwx,g=rx,o=r test.txt # -rwxr-xr--
```

### Lesson
- 리눅스는 확장자에 의미를 두지 않음 -> 확장명이 파일 종류를 결정하지 않는다
	- 해당 파일이 어떤 파일인지 알려면 file 명령어 사용

---

### Command
- 파일 소유권 변경
```bash
# chown
chown rocky test.txt # 소유자를 rocky로 변경
chown rocky.rocky test.txt # 소유자 rocky, 그룹 rocky로 변경

# chgrp
chgrp rocky test.txt # 그룹을 rocky로 변경
```

### Lesson
- 파일 소유권은 파일을 소유한 사용자와 그룹을 의미

## 파일, 디렉터리의 소유와 허가권 - 실습
### Command
```bash
[root@localhost ~] echo "ls /var" > test
[root@localhost ~] ls -l test
[root@localhost ~] ./test

[root@localhost ~] chmod 775 test
[root@localhost ~] ls -l test
[root@localhost ~] ./test

[root@localhost ~] chown rocky test
[root@localhost ~] chgrp rocky test
[root@localhost ~] su rocky

[rocky@localhost ~] pwd
[rocky@localhost ~] ls -l /root/test
[rocky@localhost ~] ls -ld /root

[root@localhost ~] mv test ~rocky

[rocky@localhost ~] chmod 777 test
[rocky@localhost ~] ls -l test
[rocky@localhost ~] chown root.root test
```

### Result
```bash
# ls-l test
-rw-r--r-- 1 root root 8  9월 29 10:56 test

# ./test
bash: ./test: 허가 거부

# chmod 775 test, ls -l test
-rwxrwxr-x 1 root root 8  9월 29 10:56 test

# ./test
account  cache	db     ftp    kerberos	local  log   nis  preserve  spool  yp
adm	 crash	empty  games  lib	lock   mail  opt  run	    tmp

# chown rocky test
-rwxrwxr-x 1 rocky root 8  9월 29 10:56 test

# chgrp rocky test
-rwxrwxr-x 1 rocky rocky 8  9월 29 10:56 test

# 사용자 전환
# pwd, ls -l, ls -l
/home/rocky
ls: cannot access '/root/test': 허가 거부 
dr-xr-x---. 15 root root 4096  9월 30 09:57 /root # root 디렉터리는 other 권한이 없음 따라서 접근 불가

# 이후 rocky로 test파일을 이동시킨 후 권한 부여 후 실행됨을 확인
# chown root.root test
chown: changing ownership of 'test': 명령을 허용하지 않음
```

### Lesson
- `./test` - 현재 디렉터리의 test 파일을 실행
- 사용자 변경
	- `su - rocky` - 로그인 환경으로 전환, rocky 사용자의 홈 디렉토리로 이동
		- `[rocky@localhost ~]`
	- `su rocky` - 현재 디렉토리를 유지하기에 사용자만 바꿔서 일시적인 작업하기 좋음
		- `[rocky@localhost root]`
- 파일의 소유권을 바꾸는 `chown` 명령은 root 사용자만 실행할 수 있다

## 링크

### Command
```bash
ln basefile hardlink # 하드 링크 생성
ln -s basefile softlink # 소프트 링크 생성
ls -il
```

### Result
```bash
68941451 -rw-r--r-- 2 root root 12 10월  1 00:00 basefile
68941451 -rw-r--r-- 2 root root 12 10월  1 00:00 hardlink
68941450 lrwxrwxrwx 1 root root  8 10월  1 00:01 softlink -> basefile
```

### Lesson
- 윈도우의 바로가기와 비슷한 기능
- 하드 링크
	- 원본 파일과 동일한 inode를 직접 가리킴
	- 원본 파일이 없어져도 이상이 없음
- 심볼릭 링크
	- 원본 파일과 다른 inode를 가리킴
	- 원본 파일이 없어지면 연결이 끊김
- inode
	- 리눅스/유닉스의 파일 시스템에서 파일, 디렉토리의 메타데이터를 저장하는 데이터구조
		- 파일의 허가권, 소유권, 위치 등
	- 모든 파일, 디렉터리는 고유한 inode를 가짐
		- inode는 실제 데이터 블록의 위치를 가리킴, 파일이름은 별칭
	- inode가 모여 있는 공간은 inode 블록
	- `ls -li` - inode번호 + 상세정보, `stat` - inode의 모든 정보
	- https://light-tree.tistory.com/300
	- https://btcd.tistory.com/1116

## 리눅스 관리자를 위한 명령어

### Command
- DNF 패키지 설치 실습
```bash
dnf -y install 패키지이름 # 기본 설치 방법
dnf install rpm파일이름.rpm # rpm 파일 설치 방법

dnf check-update # 업데이트 가능한 목록 확인
dnf update 패키지이름

dnf remove 패키지이름
dnf info 패키지이름

# mysql-errmsg 설치
dnf info mysql-errmsg # 설치할 패키지 정보 확인해보기
dnf install mysql-errmsg # 의존성이 있는 패키지까지 함께 설치

dnf -y remove 패키지이름
```

### Result
```bash
# dnf install mysql-errmsg
마지막 메타자료 만료확인(0:01:28 이전): 2025년 10월 01일 (수) 오전 12시 26분 25초.
종속성이 해결되었습니다.
================================================================================
 꾸러미                        구조       버전               저장소        크기
================================================================================
설치 중:
 mysql-errmsg                  aarch64    8.0.43-1.el9_6     appstream    498 k
종속 꾸러미 설치 중:
 mariadb-connector-c-config    noarch     3.2.6-1.el9_0      appstream    9.8 k
 mysql-common                  aarch64    8.0.43-1.el9_6     appstream     68 k

연결 요약
================================================================================
설치  3 꾸러미

전체 내려받기 크기: 576 k
설치된 크기 : 10 M
진행할까요? [y/N]: 
```

### Lesson
- RPM (redhat package manager)
	- 의존성 문제가 많아 이를 해결하고자 등장한 것이 DNF

---

### Command
- DNF 고급
```bash
dnf groupinstall "패키지그룹이름" # 패키지 그룹에 포함되는 패키지 전체 설치
dnf list httpd* # 제공되는 패키지 리스트 표시, httpd이름이 들어간 패키지 목록 표시
dnf provides httpd # 특정 파일이 어느 패키지에 있는지 확인
dnf install --nogpgcheck rpm파일이름.rpm # 인증되지 않은 rpm 패키지 GPG 인증 스킵하고 설치
dnf clean all # 다운로드한 패키지 캐시 제거
```

### Lesson
- `dnf groupinstall` 명령의 경우 큰 따옴표 안에 써야한다
- `dnf clean all` 패키지를 제거하는 것이 아닌 설치/업데이트를 위한 공간을 정리하는 명령
	- 설치 및 업데이트시 사용한 임시 파일 제거
	- 일반적으로 `/etc/yum.repos.d/` 폴더의 저장소 목록 내용을 변경한 후 실행

## DNF의 작동 방식과 설정 파일

### Lesson
- dnf 명령과 관련된 설정 파일은 `/etc/yum.conf` 와 `/etc/yum.repos.d` 에 있음
	- `/etc/yum.repos.d/`의 파일에는 패키지 파일을 검색하는 네트워크의 주소가 존재
- `dnf install` 명령 작동 순서
	1. `dnf install`
	2. `/etc/yum.repos.d/` 디렉터리의 repo 파일을 열어 URL 주소 확인
	3. URL에 접속하여 **메타데이터**(패키지 목록, 버전, 의존성 정보가 담긴 파일)를 요청
	4. 다운로드한 메타데이터를 기반으로, 설치할 패키지가 필요로 하는 다른 모든 패키지들을 연쇄적으로 찾아내 최종 설치 목록을 확정
	5. 의존성 계산이 끝난 최종 설치 목록을 보여주고 설치 여부 확인
	6. y를 입력하면, 설치 목록에 있는 모든 패키지의 실제 파일(`.rpm`)을 해당 저장소에서 다운로드
	7. 다운로드한 패키지들의 유효성을 검증한 후, 시스템에 순서대로 설치
- `gpgcheck`
	- 패키지의 GPG 서명 확인 진행 여부이며 1(사용), 0(미사용)
		- 1로 지정시 gpgkey를 작성해야 한다
		- GPG 서명은 정상적인 패키지임을 인증할 때 사용되는 암호화된 서명

## DNF 고급 실습
### Command
```bash
dnf grouplist # 패키지 그룹 목록 확인
dnf grouplist hidden > glist.txt # 모든 그룹 패키지를 표시하여 glist에 저장

dnf groupinstall "Java Platform"
```

### Lesson
- `chvt 4`
	- 가상콘솔 이동 명령어

## 파일 압축과 묶기
### Command
- 파일 압축
```bash
# bzip2, gzip 과 명령어 동일
xz sample.txt # sample.txt를 sample.txt.xz로 압축
xz -d sample.txt # sample.txt.xz 압축해제

zip test.zip sample.txt # test.zip으로 압축
unzip sample.txt.zip # 압축해제

```

### Lesson
- 최근엔 xz나 bzip2를 사용하는 추세
	- 압축 후 압축 대상 파일은 삭제된다
		- 삭제를 방지하려면 `xz -k sample.txt` -k 옵션 주기 (keep)
		- `zip`은 압축 대상 파일 삭제 x

### Command
- 파일 묶기
```bash
tar cvf my.tar /etc/sysconfig/ # 묶기
tar cvfJ my.tar.xz /etc/sysconfig # 묶기 + xz로 압축

tar tvg my.tar # 파일 확인
tar xvf my.tar # tar 풀기
tar cxvf newdir my.tar # newdir에 풀기

tar xfJ my.tar.xz # xz 압축 해제 + tar 풀기
```

### Lesson
- 윈도우는 2개의 파일을 압축할 수 있지만 리눅스는 파일 압축과 파일 묶기가 별개임
	- tar 사용
- tar 동작
	- c - 새 묶음 파일 생성
	- x - 묶음 파일 해제
	- t - 묶음 파일 해제 전 묶인 경로 표시
	- C - 지정된 디렉터리에 묶음 파일 해제, 지정하지 않으면 묶을 때와 동일한 디렉터리에
- tar 옵션
	- f(필수) - 묶음 파일의 이름 지정, 생략시 테이프로 보냄
	- v - 파일이 묶이거나 풀리는 과정 표시
	- J - tar+xz
	- z - tar+gzip
	- j - tar+bzip2

## 파일 위치 검색
### Command
```bash
find /etc -name "*.conf" # /etc 디렉터리 하위의 확장명이 conf인 파일 검색
which ls# 실행 파일의 전체 경로 찾아줌
whereis ls# 실행 파일 및 파일의 위치 (더자세함)
locate anaconda-ks.cfg # 파일 데이터베이스를 검색하여 찾아줌
```

## 시스템 설정
### Command
```bash
nmtui # 네트워크 설정
firewall-config # 방화벽 설정
ntsysv # 서비스(데몬) 설정
```

## 새로 배운 내용
### Lesson
- `grub` 는 무엇일까?
	- 해상도 설정하며 계속 보이던 건데 궁금했음
	- GRUB 부트로더의 설정 파일들이 있는 곳
	- GRUB - GRand Unified Bootloader 리눅스 시스템이 부팅될 때 사용하는 부트로더
	- 메인 설정 파일이며 부팅 옵션, 타임아웃, 기본 OS 등을 설정함
- repo의 baseos, appstream ... 은 무엇일까?
	- baseos - 핵심 구성요소, 부팅 후 최소한의 기능을 수행하는 데 필요한 필수 패키지가 담긴 리포지토리
	- appstream - 운영체제 위에서 사용자가 활용하는 소프트웨어, 프로그래밍 언어 등을 제공하는 리포지토리
	- extras - 배포판 개발팀이 제공하고 유지보수하는 추가 패키지를 모아놓은 리포지토리
	- plus - baseos에 포함된 핵심 시스템 구성 요소를 대체하거나 업그레이드하는 패키지를 제공하는 리포지토리
	- crb - 다른 소프트웨어를 소스 코드로부터 빌드하거나 개발하는 데 필요한 도구와 라이브러리를 제공하는 리포지토리