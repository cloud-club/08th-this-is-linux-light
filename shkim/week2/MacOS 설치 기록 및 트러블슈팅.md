## 네트워크 설정
- `ifconfig` 명령어를 통해 내 ip주소를 확인하기
- VMware Fusion - preferences - network - custom(추가)
	- Allow using NAT / Subnet IP를 `192.168.111.0`
- 설치된 가상머신 - settings - Network Adapter - 추가한 custom으로 바꿔주기

## Linux 설치
- 9.6버전에는 소프트웨어 선택 - 워크스테이션이 없음
	- GUI를 갖춘 서버로 설치 후 워크스테이션 그룹 추가 설치하면 된다
		- `sudo dnf groupinstall "Workstation"`
		- 만약 워크스테이션 환경이 필요없다면 그냥 GUI를 갖춘 서버 선택
			- **이게 표준임 그래픽환경 원하면**

## 트러블슈팅
### Error
```bash
# dnf makecache

Rocky Linux 9 - BaseOS                          109  B/s | 153  B     00:01    
Errors during downloading metadata for repository 'baseos':
  - Status code: 403 for https://dl.rockylinux.org/vault/rocky/9.6/BaseOS/aarch64/os/repodata/repomd.xml (IP: 146.75.94.132)
오류: repo를 위한 메타자료 내려받기에 실패하였습니다 'baseos': Cannot download repomd.xml: Cannot download repodata/repomd.xml: All mirrors were tried
```

```bash
# x86_64로 했을 경우에 발생하는 에러
 문제: 충돌하는 요청
  - package mc-1:4.8.26-5.el9.x86_64 from appstream does not have a compatible architecture
(설치 할 수 없는 꾸러미를 건너 뛰려면 '--skip-broken'을 (를) 추가하십시오 또는 '--nobest'는 최적 후보의 꾸러미만 사용합니다)
```
### Solve
```
[baseos]
name=Rocky Linux $releasever - BaseOS
baseurl=https://download.rockylinux.org/pub/rocky/9.6/BaseOS/aarch64/os/
gpgcheck=0

[appstream]
name=Rocky Linux $releasever - AppStream
baseurl=https://download.rockylinux.org/pub/rocky/9.6/AppStream/aarch64/os/
gpgcheck=0

[extras]
name= Rocky Linux $releasever - Extras
baseurl=https://download.rockylinux.org/pub/rocky/9.6/extras/aarch64/os/
gpgcheck=0

[plus]
name= Rocky Linux $releasever - Plus
baseurl=https://download.rockylinux.org/pub/rocky/9.6/plus/aarch64/os/
gpgcheck=0

[crb]
name=Rocky Linux $releasever - CRB
baseurl=https://download.rockylinux.org/pub/rocky/9.6/CRB/aarch64/os/
gpgcheck=0
```

### Lesson
- 현재 설치한 버전이 9.6이므로 실습 파일(9.0)과 조금 다름
- `/vault`는 이전 버전을 보관하는 아카이브, 403 발생
	- 따라서 `/pub` 으로 경로 설정
- 아키텍처도 맞춰줘야한다