
## 네트워크 관련 설정과 명령어
### TCP/IP
- 인터넷에서 컴퓨터들이 통신하기 위한 기본 프로토콜 모음

### 호스트 이름과 도메인 이름
- **호스트 이름**은 네트워크에 연결된 각 컴퓨터(장치)에 부여된 고유한 이름
- **도메인 이름**은 숫자로 된 IP 주소를 사람이 기억하기 쉬운 문자 형태의 주소
- `hostname.example.com`
	- `hostname`은 호스트 이름 `example.com`은 도메인 이름

### IP
- 각 컴퓨터의 랜 카드에 부여되는 중복되지 않는 유일한 주소

### 네트워크 주소
- 같은 네트워크에 속해 있는 장치가 공유하는 공통 주소
- `192.168.x.x`

### 브로드캐스트 주소
- 같은 네트워크에 있는 모든 장치에 한 번에 데이터를 전송할 수 있는 주소
- `192.168.x.255`

### 게이트웨이
- 내부 네트워크를 외부 네트워크에 연결할 때 필요한 장치
- 인터넷으로 접속할 때 반드시 거쳐가야 하는 지점의 IP 주소
- `route add default gw 192.168.111.254 dev ens160`
	- 게이트웨이 주소 변경 명령어

### 넷마스크와 클래스
- 넷마스크는 IP 주소에서 네트워크 부분과 호스트 부분을 구분하는 역할
	- `255.255.255.0`은 네트워크 주소로 앞 3자리를 사용하고 호스트 주소로 마지막 1자리를 사용한다는 뜻
- 클래스는 과거 IP 주소 대역을 나누던 방식 (A ~ E 클래스)
	- 현재는 CIDR 표기법을 사용함
		- [[01. 사설망과 CIDR]]

### DNS 서버 주소
- 도메인 이름을 실제 IP 주소로 변환해주는 서버의 IP 주소
	- DNS는 URL을 IP 주소로 변환해주는 시스템

### 명령어
```bash
nmtui # 네트워크 관련 작업 처리
systemctl start NetworkManager # 시스템 및 서비스 관리를 위해 사용하는 명령어
ifconfig # ip 주소와 관련 정보
nslookup # dns서버 작동 테스트
ping # 네트워크 응답 테스트
```

### 주요 파일
```bash
# ens160 장치에 설정된 네트워크 정보가 있는 파일
/etc/NetworkManager/system-connection/ens160.nmconnection

# DNS 서버의 정보와 호스트 이름이 있는 파일
/etc/resolv.conf

# 현재 컴퓨터의 호스트 이름과 FQDN이 있는 파일
/etc/hosts
```

### Lesson
- `nmtui` 또는 `ens*.nmconnection` 통해 네트워크 관련 사항을 설정한 뒤 `systemctl restart NetworkManager` 명령을 입력해야 변경한 내용이 적용된다.
- `8.8.8.8`은 구글의 공개 DNS 서버 주소
- `naver.com` 접속 과정
	- DNS 캐시 확인 (해당 url의 ip가 존재하는지)
	- DNS 쿼리 전송 (8.8.8.8에 naver.com ip주소 요청)
	- DNS 응답 (8.8.8.8이 naver.com의 ip를 찾아서 사용자에게 응답을 보냄)
	- naver.com ip로 접속

---

## SELinux

### Command
```bash
sestatus # SELinux 활성화 여부 확인

# SELinux 활성화 및 비활성화
grubby --update-kernel ALL --args selinux=0 # 비활성화
grubby --update-kernel ALL --remove-args selinux # 활성화
```

### Lesson
- SELinux 활성화 시 기본적으로 enforcing(강제) 모드로 설정된다.
	- permission(허용)모드로 설정하려면 `/etc/sysconfig/selinux` 편집
		- `system-config-selinux`으로도 가능
- enforcing(강제) - 시스템 보안에 영향을 미치는 기능이 감지되면 **해당 기능이 작동하지 않도록 시스템에서 막는** 모드
- permission(허용) - 시스템 보안에 영향을 미치는 기능이 감지되면 **허용은 하되 사용 내역이 로그에 남고 화면상에 출력**되는 모드

---

## 파이프, 필터, 리디렉션

### Command
```bash
# 파이프 -> |
ls -l /etc | more # 페이지를 나눠서 보겠다는 의미

# 필터
ps -ef | grep bash # bash 글자가 들어간 프로세스만 출력

# 리디렉션
ls -l > list.txt # list.txt 파일을 만들고 ls -l 명령어의 결과를 파일에 저장
ps aux | grep sshd >> list.txt # 기존 내용을 덮어쓰지 않고 결과를 파일에 저장
sort < list.txt # list.txt 파일을 정렬해서 화면에 출력
sort < list.txt > out.txt # list.txt 정렬 후 out.txt 파일에 저장
```

## 프로세스

### Command
```bash
ps # 현재 프로세스의 상태를 확인하는 명령
kill 3066 # 프로세스 종료 (프로세스에 요청)
kill -9 3066 # 프로세스 강제 종료 (커널 단위)
pstree # 부모 프로세스와 자식 프로세스의 관계를 트리 형태로 출력
```

### Lesson
- 하드디스크에 저장된 실행 코드(프로그램)가 메모리에 로딩되어 활성화된 것
- **Foreground Process(포그라운드 프로세스)**
	- 프로그램이 화면에 나타나 사용자와 상호작용 하는 프로세스
- **Background Process(백그라운드 프로세스)**
	- 실행은 되었지만, 화면에는 나타나지 않는 프로세스
	- 백신 프로그램, 서버 데몬
- **PID(프로세스 번호)**
	- 각각의 프로세스에 할당된 고유 번호
- 작업 번호
	- 실행 중인 백그라운드 프로세스의 순차 번호
- 부모 프로세스와 자식 프로세스
	- **모든 프로세스는 부모 프로세스에 종속되어 실행된다.**
	- X윈도를 강제 종료하면 Firefox도 함께 종료된다.

## 프로세스 실습
### Command
- **포그라운드, 백그라운드 프로세스 확인 및 종료** 

```bash
yes > /dev/null # yes를 계속 출력하는 프로세스 생성 (포그라운드)
# ctrl+z 로 일시정지

bg # 백그라운드로 프로세스 실행
ps -ef | grep yes # yes 프로세스 확인
jobs # 백그라운드로 가동 중인 프로세스 확인
fg 1 # 포그라운드로 프로세스 실행

kill -9 프로세스번호 # 프로세스 강제 종료
```

### Lesson
- `/dev/null`으로 보내진 데이터는 어디에도 저장되지 않고 바로 사라진다.
	- 아무것도 아닌 장치, 따라서 `y`출력이 전부 버려짐
- 백그라운드로 프로세스를 실행하려면 뒤에 `&`을 붙인다.

---

## 서비스와 소켓

### Command
```bash
systemctl start/stop/restart 서비스이름 # 서비스 시작/중지/재실행
systemctl status # 서비스 상태 확인
systemctl enable/disable 서비스이름 # 서비스 사용/사용 안 함

ls -l /usr/lib/systemd/system | more # 기본으로 제공되는 서비스 파일 확인

systemctl list-unit-files # 서비스 자동 실행 여부 확인
```

### Lesson
**서비스와 소켓**
- 서비스는 서버 프로세스를 의미하며 데몬이라고도 불린다.
	- 웹 서버, 네임 서버 등의 프로세스 or 웹 서버 데몬
	- 백그라운드 프로세스의 일종이다.
	- 서비스의 실행 스크립트 파일명은 `서비스이름.service`
- 소켓은 필요할 때만 작동하는 서버 프로세스를 의미한다.
	- 요청 시 `systemd`가 소켓을 구동, 요청이 종료되면 소켓 종료
	- 소켓과 관련된 스크립트 파일명은 `소켓이름.socket`

**추가 개념**
- `systemd`는 시스템이 부팅될 때 커널에 의해 가장 먼저 실행되는 프로세스이며, 다른 모든 시스템 프로세스를 시작하고 관리하는 역할 ![[Pasted image 20251010080435.png]]
- `systemctl`은 `systemd` 시스템 및 서비스 관리자를 제어하고 상태를 확인하기 위한 명령어.
- `systemctl list-unit-files`
	- static -> 다른 서비스나 소켓에 의존해서 실행 따라서 자동 실행 여부 설정 불가.
	- generated -> 시스템 부팅 시점에 자동으로 생성된 유닛
	- transient -> 런타임 중에 사용자의 요청으로 임시로 생성된 유닛

---

## 응급 복구

### Command
- **root 계정 까먹었을 때**

```bash
# 부팅 시 GRUB 부트 로더가 나타나면 E를 눌러 편집
# rhgb quiet 을 지우고 끝에 init=/bin/sh 입력
... selinux=0 init=/bin/sh
# ctrl+x 재부팅

# sh5-1# 프롬프트가 나온다
mount # 하지만 / 파티션이 읽기 전용임
mount -o remount,rw / # / 파티션을 읽기/쓰기 모드로 재마운트한다.

passwd # 비밀번호 재설정
```

### Lesson
- 이 복구 방법이라면 누구나 root 권한을 얻을 수 있지 않나?
	- **따라서 GRUB 부트 로더를 편집할 수 없도록 설정해야 한다.**

---

## GRUB 부트 로더

### Command
- **GRUB 부트 로더를 편집할 수 없도록 설정**

```bash
vi /etc/default/grub # GRUB_TIMEOUT=20, 운영체재 선택 대기 시간 20초로
grub2-mkconfig -o /boot/grub2/grub.cfg # 변경사항 적용

vi /etc/grub.d/00_header # 부트 로더에 비밀번호 설정하는 작업 아래 입력
# cat << E0F
# set superusers="thisislinux"
# password thisislinux password
# E0F
grub2-mkconfig -o /boot/grub2/grub.cfg # 변경사항 적용
```

### Lesson
- 운영체제 커널을 메모리에 로드하여 부팅을 시작
- 설정 파일은 `/boot/grub2/grub.cfg` **단 이 파일을 직접 수정하면 안된다.**
	- 수정을 위해서는 `/etc/default/grub` 파일과 `/etc/grub.d` 파일을 수정한 후 `grub2-mkconfig` 명령으로 변경사항을 적용

## GRUB 부트 로더 트러블슈팅

### Error
![[Pasted image 20251010084231.png]]
grub 부트로더 편집할 수 없도록 `/etc/grub.d/00_header` 파일에 아래와 같은 내용을 추가:
```bash
cat << E0F
set superusers="thisislinux"
password thisislinux password
E0F
```

하지만 접근이 거부되었다는 오류 발생

### Solve
**원인**
- 해당 파일이 다른 사용자가 읽을 수 있는 권한을 가져 부팅 거부
	- `-rwxr-xr-x 1 root root 9421 10월 10 08:39 /etc/grub.d/00_header`

**해결**
- 소유자만 읽기/쓰기가 가능하도록 변경하자 
```bash
chmod 600 /etc/grub.d/00_header
grub2-mkconfig -o /boot/grub2/grub.cfg
```

---

## 커널 컴파일


### Lesson
- 커널은 하드웨어 제어
- 모듈은 필요할 때 사용하는 코드
- `;` 세미콜론으로 명령을 순차적으로 실행할 수 있다. (성공 관계 없이 순차적)
	- `cd /tmp; ls -l`
- `&&`, `||` 논리 연산자로도 순차적으로 실행 가능 (논리에 맞게 실행됨)