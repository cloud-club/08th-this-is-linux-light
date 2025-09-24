# 1~11
## 실습 트러블슈팅
### Error
- Failed to find a suitable stage1 device: EFI 시스템 파티션은 유형 swap이 될 수 없습니다.; EFI 시스템 파티션은 유형 xfs이 될 수 없습니다.; EFI 시스템 파티션은 /boot/efi 중 하나에 적재되어야만 합니다.
### Solve

- 수동 파티션 설정 시 `/boot/efi` 200MiB 할당 후 `/` , `swap` 설정
	- EFI 시스템 파티션은 `/boot/efi` 마운트 포인트를 가져야 함
	- 이후 실습을 위해 이 방법을 선택
- VMWare Workstation Pro 17 버전 사용하기

### Lesson
- EFI 시스템 파티션(ESP)이란?
	- UEFI 펌웨어를 사용하는 컴퓨터가 OS를 부팅하기 위해 사용하는 파티션
		- 부팅을 위해 먼저 읽어 들이는 공간
	- 용량은 200~600MB
- 마운트 포인트란?
	- 파티션, USB 등을 리눅스 시스템의 특정 디렉터리에 연결하는 지점
	- `/boot/efi`
		- ESP가 해당 경로에 마운트 되며 리눅스는 해당 마운트 포인트를 통해 접근

---

## 단축키 정리
+ ctrl + alt -> 마우스 포인터 꺼내기
+ shift + space -> 한/영 전환
+ ctrl + alt + del 대신 insert
+ Ctrl + Alt + F1 ~ F6 -> 가상 콘솔 이동
---

# 12~16
## 시작과 종료
### Command
```bash
# 시스템 종료 명령
poweroff
shutdown -P now
halt -p
init 0

# 시스템 재부팅
reboot
shutdown -r now
init 6

# 로그아웃
logout
exit
```

### Lesson
- 명령어 입력 시 대소문자 구분을 명확히 해야함
- `shutdown`은 명령어의 옵션을 이용해 예약된 작업을 할 수 있다
	- 예: `shutdown -P +10` - 10분 뒤 종료
- `#`은 root 사용자, `$`은 일반 사용자
- 일반 사용자가 root 권한을 얻으려면 `su-` 혹은 `su` 명령어 실행 후 root id/pw 입력

---
### Command
```bash
# 여러 명의 사용자가 동시에 리눅스에 접속해 있을 때 시스템 종료 확인

# root - tty3
shutdown -h +5 # 5분뒤 종료
```

### result
```bash
# rocky - tty4
Broadcast message from root@localhost on tty3 (date)
The system is going down for poweroff at (date)
```

### Lesson
- `shutdown -c` 예약된 shutdown 명령 취소

---

### Command
```bash
# 사용자가 로그아웃 하도록 유도하는 명령

# root - tty3
shutdown -k +5 
```

### Result
```bash
# rocky - tty4
Broadcast message from root@localhost on tty3 (date)
The system is going down for poweroff at (date)
```

### Lesson
- 시스템이 종료된다는 메시지가 나오지만 내부적으로는 입력과 동시에 shutdown 명령이 취소된다 - 시스템이 실제로 종료되지 않음

## 런레벨

### Command
```bash
cd /lib/systemd/system
ls -l runlevel?.target
```

### Result
```bash
lrwxrwxrwx. 1 root root 15  5월 27  2022 runlevel0.target -> poweroff.target
lrwxrwxrwx. 1 root root 13  5월 27  2022 runlevel1.target -> rescue.target
lrwxrwxrwx. 1 root root 17  5월 27  2022 runlevel2.target -> multi-user.target
lrwxrwxrwx. 1 root root 17  5월 27  2022 runlevel3.target -> multi-user.target
lrwxrwxrwx. 1 root root 17  5월 27  2022 runlevel4.target -> multi-user.target
lrwxrwxrwx. 1 root root 16  5월 27  2022 runlevel5.target -> graphical.target
lrwxrwxrwx. 1 root root 13  5월 27  2022 runlevel6.target -> reboot.target
```

### Lesson
- `init` 명령 뒤에 붙는 숫자를 런레벨(RunLevel)이라고 부른다
- 런레벨은 시스템이 가동되는 방법을 7가지로 나눈 것
	- 0 - poweroff
	- 1 - rescue
	- 3 - 텍스트모드
	- 5 - 그래픽모드
	- 6 - 재부팅모드
- `init 0` 명령은 즉시 runlevel0으로 시스템을 전환하라는 의미
	- 마찬가지로 `init 6` 명령은 즉시 runlevel6으로 전환하라는 의미
- 과거에는 `SysVinit` 이라는 시스템을 사용해 0~6으로 상태를 관리했었음
	- 현재는 `target` 이라는 개념으로 시스템의 상태를 관리함
	- 따라서 `runlevel1.target`이 실제로는 `poweroff.target`을 가리킴

---

### Command
```bash
cd /lib/systemd/system
ls -l default.target
```

### Result
```bash
lrwxrwxrwx. 1 root root 16  5월 27  2022 default.target -> graphical.target
```

### Lesson
- 기본으로 설정된 런레벨이 그래픽 모드로 부팅하도록 설정하는 `graphical.target` 을 가리키고 있음

---

### Command
```bash
# default.target이 가리키는 파일을 텍스트 모드로 부팅하는 런레벨로 변경하기
ln -sf /usr/lib/systemd/system/multi-user.target /etc/systemd/system/default.target
ls -l /etc/systemd/system/default.target
```

### Result
```bash
lrwxrwxrwx 1 root root 41  9월 22 22:27 /etc/systemd/system/default.target -> /usr/lib/systemd/system/multi-user.target
```

### Lesson
- 텍스트 모드로 부팅되도록 런레벨을 변경
- 텍스트 모드 상태에서 X윈도를 실행하려면 `startx` 입력

## 장치 연결 (마운트)
### Command
```bash
# 마운트된 장치 확인
mount
```

### Result
```bash
...
/dev/sda3 on / type xfs (rw,relatime,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sda1 on /boot/efi type vfat (rw,relatime,fmask=0077,dmask=0077,codepage=437,iocharset=ascii,shortname=winnt,errors=remount-ro)
...
```

### Lesson
- 물리적인 장치를 특정한 위치(디렉터리)에 연결하는 과정을 마운트라함
- 가상머신 설치할 때 파티션 나눠둔 거 여기서 볼 수 있음
- 마운트 해제 - `umount /...`

---

### Command
```bash
# 가상머신의 CD/DVD에 iso 파일을 연결한 상황
mount
```

### Result
```bash
/dev/sr0 on /run/media/root/Rocky-9-0-x86_64-dvd ...
```

### Lesson
- CD/DVD 장치인 `/dev/sr0`이 `/run/media/...`에 자동 마운트 되었음

---

### Command
```bash
# 마운트된 디렉토리로 이동
cd /run/media/root/Rocky-9-0-x86_64-dvd/
pwd
ls

# DVD 안에 있는 파일 확인
cd BaseOS/Packages
ls
cd a
ls
```

### Result
```bash
# pwd
/run/media/root/Rocky-9-0-x86_64-dvd

# ls
AppStream          Contributors  LICENSE                      images
BaseOS             EFI           RPM-GPG-KEY-Rocky-9          isolinux
COMMUNITY-CHARTER  EULA          RPM-GPG-KEY-Rocky-9-Testing  media.repo

# cd BaseOS/Packages , ls
a  b  c  d  e  f  g  h  i  j  k  l  m  n  o  p  q  r  s  t  u  v  w  x  y  z

# cd a , ls
accel-config-3.4.2-2.el9.i686.rpm
accel-config-3.4.2-2.el9.x86_64.rpm
accel-config-libs-3.4.2-2.el9.i686.rpm
accel-config-libs-3.4.2-2.el9.x86_64.rpm
...
```

### Lesson
- 알파벳 순으로 정렬된 각 디렉터리에는 rpm 파일이 존재
- Rocky Linux를 설치할 때 함께 설치된 것

---

### Command
```bash
# DVD를 더이상 사용하지 않으므로 마운트 해제
umount /dev/cdrom
```

### Lesson
- `/dev/cdrom`은 `/dev/sr0`에 링크된 파일이므로 사실상 동일함
	- `/dev/cdrom` 이라는 링크 이름은 대부분의 리눅스에서 대부분 동일하게 지정됨
	- `/dev/sr0`는 리눅스 종류나 버전에 따라서 다양한 이름으로 지정될 수 있음
	- 따라서 `/dev/cdrom`을 기억하는 것이 편리
- 마운트된 경로에서 `umount`시 "target is busy"라는 에러메시지가 출력된다 다른 경로에서 실행하기

---

### Command
```bash
mkdir /media/cdrom # CD/DVD 마운트하는 디렉터리 생성
mkdir /media/usb # USB 마운트하는 디렉터리 생성
mount /dev/cdrom /media/cdrom # CD/DVD 마운트
mount /dev/sdb1 /media/usb # USB 마운트

# 마운트 되었는지 확인
mount

# 이후 마운트해제
umount /media/cdrom # umount /dev/cdrom 도 가능
umount /media/usb
```

### Result
```bash
# 마운트 되었는지 확인
/dev/sr0 on /media/cdrom type ...
/dev/sdb1 on /media/usb type ...
```

### Lesson
- 텍스트 모드에서는 CD/DVD를 넣거나 USB메모리를 연결해도 자동으로 마운트되지 않는다 
- USB 가상머신에 연결시 해당 메모리가 어디에 연결되었는지 로그 발생
	- 나의 경우 `sdb: sdb1`
- 호스트 컴퓨터와 가상머신 간 USB를 통해 파일을 주고받을 수 있다다

## 리눅스 기본 명령
### Command
```bash
ls # 디렉터리에 있는 파일의 목록 나열
cd # 디렉터리 이동
pwd # 현재 디렉터리 경로 출력
rm # 파일이나 디렉터리 삭제
cp # 파일이나 디렉터리 복사
touch # 크기0 새 파일 생성 혹은 존재한다면 최종 수정 시간 변경
mv # 파일 이동 / 파일명 변경 후 이동 가능
mkdir # 디렉터리 생성
rmdir # 디렉터리 삭제
cat # 파일 내용 화면 출력
head, tail # 텍스트 형식으로 작성된 파일의 앞 또는 마지막 행 화면 출력
more # 텍스트 형식으로 작성된 파일을 페이지 단위로 화면에 출력
less # more 보다 더 기능이 다양한
file # 파일의 종류 표시
clear # 터미널 화면 청소
```

### Lesson
- 파일 이름이나 디렉터리의 맨 앞 글자를 `.`으로 지정하면 숨김 파일/디렉터리가 된다