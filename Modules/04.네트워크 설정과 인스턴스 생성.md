# 개요

이번 학습 모듈에서는 NHN Cloud에서 Linux 기반 웹 서버를 생성해 원격 접속하여 구동하는 방법을 다룹니다. NHN Cloud는 사용자 친화적인 인터페이스와 다양한 클라우드 리소스를 통해 누구나 쉽게 안정적이고 효율적인 IT 환경을 구축할 수 있도록 지원합니다.

# 학습 목표

이번 학습 모듈에서 배울 내용은 다음과 같습니다. 

* **인스턴스 생성**
    * Ubuntu 서버를 기반으로 한 Linux 인스턴스 생성
    * 기본적인 서버 환경을 구축
    * 인스턴스 유형, 키페어 설정, 보안 그룹 및 네트워크 설정을 포함한 세부 구성
* **네트워크 설정**
    * VPC, 서브넷, 라우팅 테이블 설정
* **SSH 원격 접속**
    * 생성된 인스턴스에 SSH를 통해 원격 접속
* **웹 서버 설치 및 실행**
    * 웹 서버 설치 및 구동하기

![모듈4시나리오구성도_rv1](https://github.com/user-attachments/assets/6a170fa3-ff3a-4211-b33f-d08bb276433a)

# 시작하기 전에

NHN Cloud를 시작하기 위해서는 다음 사항을 준비해야 합니다.

* **표준 인터넷 브라우저**
    * Google Chrome, Microsoft Edge, Firefox, Safari 등의 최신 버전 브라우저가 설치되어 있어야 합니다.
    * 브라우저 설정에서 JavaScript 및 쿠키가 활성화되어 있어야 합니다.
* **인터넷 연결 환경**
    * 안정적인 인터넷 연결이 필요하며, 권장 대역폭은 최소 5Mbps 이상입니다.
    * HTTPS를 통한 안전한 통신이 가능해야 합니다.
* **회원 계정**
    * 결제수단을 등록한 NHN Cloud 계정이 있어야 합니다.
    * NHN Cloud 포털에 로그인 해야 합니다.

!!! tip "알아두기"
본 가이드는 [03-IAM 계정과 거버넌스 설정](dooray://1387695619080878080/pages/3977493217025617647 "publish") 이후 단계부터 시작합니다.

# 인스턴스 생성을 위한 준비

## 단계 1. 기본 인프라 서비스 활성화하기

1. NHN Cloud 콘솔에 접속한 후, 상단 메뉴에서 실습에 사용할 조직(`MyORG`), 프로젝트(`MyPRJ`), 그리고 `한국(평촌) 리전`을 선택했는지 확인합니다.
2. `MyPRJ` 오른쪽에 위치한 **서비스 선택**을 클릭합니다.
3. **서비스 선택** 클릭한 뒤 **Compute - Instance**를 클릭합니다.
4. **서비스 활성화** 창에서 **확인**을 클릭합니다.

!!! tip "알아두기"
    * 서비스 활성화
        * 서비스가 활성화하면 NHN Cloud 콘솔 프로젝트 화면 왼쪽에 서비스 메뉴가 노출됩니다. 해당 메뉴에서 필요한 리소스를 관리할 수 있습니다.
        * 설정한 리전에 따라 일부 서비스는 제공되지 않을 수 있습니다.
        * Instance 서비스를 활성화하면 관련된 기본 인프라 서비스가 함께 활성화됩니다.

## 단계 2. 기본 네트워크 설정

> 인스턴스 원격 접속에 필요한 기본적인 네트워크 리소스 VPC, 서브넷, 라우팅 테이블의 이름을 설정합니다.

1. 왼쪽 메뉴에서 **Network - VPC**를 클릭합니다.
2. `Default Network` (CIDR: 192.168.0.0/16)를 선택 후 **VPC 변경**을 클릭합니다.
3. **VPC 변경** 창에서 아래 내용을 입력 후 **확인**을 클릭합니다.
    * 이름: `MyVPC`
4. 성공 창에서 **확인**을 클릭합니다.
5. 왼쪽 메뉴에서 **Network - Subnet**을 클릭합니다.
6. `Default Network` (CIDR: 192.168.0.0/24)를 선택 후 **서브넷 변경**을 클릭합니다.
7. **서브넷 변경** 창에서 아래 내용을 입력 후 **확인**을 클릭합니다.
    * 이름: `MySubnet`
8. 성공 창에서 **확인**을 클릭합니다.
9. 왼쪽 메뉴에서 **Network - Routing**을 클릭합니다.
10. vpc-(임의 이름 값) 을 선택 후 **라우팅 테이블 변경**을 클릭합니다.
11. **라우팅 테이블 변경** 창에서 아래 내용을 입력 후 **확인**을 클릭합니다.
    * 이름: `MyRT`
12. 성공 창에서 **확인**을 클릭합니다.

## 단계 3. Linux 인스턴스 생성하기

1. 왼쪽 메뉴에서 **Compute - Instance**를 클릭합니다.
2. **인스턴스 생성**을 클릭합니다.
3. **인트턴스 생성** 창에서 아래 정보를 설정 후 **인스턴스 생성**을 클릭합니다.
    * 인스턴스 템플릿
        * 사용: `사용 안 함`
    * 이미지
        * 공용 이미지 > 이미지 이름: `Ubuntu Server 22.04 LTS` 클릭(공용 이미지 목록의 하단에서 해당 이미지를 찾을 수 있습니다.)

!!! tip "알아두기"
    * 쉬운 공용 이미지 검색 팁
        * **공용 이미지 탭** 하단에 "전체 이미지" 클릭 후 드롭다운 메뉴에서 `OS → Ubuntu`를 선택해 대상 이미지 확인 후 선택
        * **공용 이미지 탭** 하단에 "전체 이미지" 오른쪽에 있는 "이미지 이름 또는 설명을 입력해 주세요." 입력 칸에 `Ubuntu Server 22.04` 를 입력하여 대상 이미지 확인 후 선택

    * 인스턴스 정보
        * 가용성 영역: `임의의 가용성 영역`
        * 인스턴스 이름: `linux-server-basic`
        * 인스턴스 타입: **인스턴스 타입 선택** 클릭 > 인스턴스 타입 이름: `t2.c1m1` (분류: Standard, 타입: t2, 이름: t2.c1m1, vCPU: 1, 메모리: 1GB) 클릭 후 **선택** 클릭
        * 인스턴스 수: `1`
        * 키페어 > **+ 생성** 클릭 > 키페어 선택 하단에 `MyKey` 입력 후 **키페어 생성** > **키페어 다운로드** 클릭 후 `MyKey.pem` 파일 다운로드

!!! caution "주의"
    * 키페어 관리
        * 키페어 다운로드는 키페어 생성 시 한번만 다운로드 받을 수 있습니다. 파일을 안전한 곳에 다운로드하여 관리하세요.
        * 이미 생성한 `MyKey` 키페어가 "키페어 선택" 드롭다운 메뉴에 있으면 해당 키를 선택하시기 바랍니다. 단, MyKey.pem 파일을 사용자가 기억하는 경로(디렉토리 또는 폴더)에 다운로드 받아놓은 상태여야 합니다.
        * 키페어를 생성하면 자동으로 해당 키페어 값이 선택됩니다.
        * 자세한 내용은 아래 참고 자료의 키페어 사용자 가이드를 참고하세요.

    * 루트 블록 스토리지
        * 블록 스토리지 타입: `HDD`
        * 블록 스토리지 크기(GB): `20` GB
    * 네트워크 설정: `네트워크 인터페이스 생성` 선택
    * 네트워크
        * 사용 가능한 서브넷 항목에서 `MySubnet (192.168.0.0/24)` 리소스를 클릭하여 선택된 서브넷으로 사용
    * 플로팅 IP: **설정 변경** 클릭
        * 사용: `사용`
        * 삭제 보호: `사용 안 함`
        * 연결할 서브넷: `MySubnet (192.168.0.0/24)`
    * 보안 그룹
        * **보안 그룹 생성** 클릭
        * **보안 그룹 생성** 창에서 아래 정보를 설정 후 **확인**을 클릭합니다.
            * 이름: `MySG-SSH`
            * 보안 규칙 추가에  **+**을 클릭하여 다음 설정의 보안 규칙을 추가
                * 방향: `수신`
                * IP 프로토콜: `SSH`
                    * 해당 IP 프로토콜을 선택하면 자동으로 포트 정보가 입력됩니다.
                * Ether: `IPv4`
                * 원격: `CIDR - 0.0.0.0/0`
        * 성공 창에서 **확인**을 클릭
        * **보안 그룹 선택** 항목에서 위에 생성한 `MySG-SSH`를 선택
    * 추가 블록 스토리지: 사용 안 함(기본)
    * 사용자 스크립트: 빈 칸(기본)
    * 삭제 보호: 사용 안 함(기본)
4. 인스턴스 생성 정보 창에서 **인스턴스 생성**을 클릭합니다.
5. 인스턴스 생성 작업이 진행됩니다. 해당 인스턴스는 몇 분 내외로 생성이 완료됩니다.

!!! tip "알아두기"
    * 사용자 정의 TCP
        * IP 프로토콜을 "사용자 정의 TCP"로 선택하면 포트값을 사용자가 직접 입력할 수 있습니다. **1**로 표기된 입력칸에 `22`를 입력합니다.

# 인스턴스 접속 및 Nginx 웹 서버 구동

## 단계 1. SSH 원격 접속하기

1. 왼쪽 메뉴에서 **Network - Floating IP**를 클릭합니다.
2. 플로팅 IP 리소스 목록 중 연결된 장치가 `linux-server-basic`인 IP 주소를 **복사** 후 **기록**합니다.
3. 아래 사용자 환경에 맞춰 SSH 원격 접속을 합니다.

    * **Windows 사용하는 경우**
        * Windows **시작**을 클릭 후 `Windows PowerShell`을 검색하여 실행합니다.

!!! tip "알아두기"
    * Windows PowerShell 실행 팁
        * `윈도우 키(⊞) + R` 을 눌러 실행창을 실행합니다.
        * 열기(O) 입력 칸에  `powershell.exe` 를 입력 후 엔터키를 누르면 Windows PowerShell이 실행됩니다.

        * 다운로드한 키페어인 `MyKey.pem` 폴더로 이동합니다.

            ```PowerShell
            cd \(MyKey.pem 파일이 있는 폴더명)
            ```

            
        *`MyKey.pem` 권한을 초기화 합니다.

            ```PowerShell
            icacls MyKey.pem /reset
            ```
        
        * PowerShell을 실행한 사용자의 계정에 `MyKey.pem` 권한을 부여합니다.

            ```PowerShell``` 
            icacls MyKey.pem /grant:r "$($env:username):(r)"
            ```
        
        * `MyKey.pem` 파일이 상위 폴더로부터 권한을 상속받지 않도록 설정합니다.

            ```PowerShell```
            icacls MyKey.pem /inheritance:r
            ```

!!! tip "알아두기"
    SSH 키나 보안 인증서 파일은 엄격하게 권한을 제한해야 합니다. Windows에서는 icacls 명령어를 사용해 상속을 차단하고, 사용자별 읽기 권한을 부여하여 보안 수준을 유지합니다.
            
        * ssh 명령어로 인스턴스에 접속합니다.

            ```PowerShell
            ssh -i MyKey.pem ubuntu@복사한 linux-server-basic 플로팅 IP 주소
            ```
            * 복사한 IP 주소
                * 위의 단계에서 확인한 플로팅 IP 주소 입니다.

        * "Are you sure you want to continue connecting (yes/no/[fingerprint])?" 문구가 나오면 `yes` 를 입력 후 엔터키를 입력합니다.
        * lsb\_release 명령어로 현재 접속한 Linux 버전을 확인합니다.

            ```bash
            lsb_release -a
            ```
            
    * **macOS 사용하는 경우**
        * Dock에서 **터미널(Terminal)** 앱을 실행하거나, Spotlight에서 **터미널**을 검색하여 실행합니다.

!!! tip "알아두기"
    * 터미널 실행 팁
        * `Command(⌘) + Space` 를 눌러 Spotlight를 실행합니다
        * 입력 칸에  `"터미널"` 입력 후 <span style="color:rgb(51, 51, 51);">Return키</span>를 누르면 터미널이 실행됩니다.
        * 다운로드 받은 키페어인 `MyKey.pem` 디렉터리로 이동합니다.

            ```bash
            cd /(MyKey.pem 파일이 있는 디렉터리명)
            ```

            
        * 아래 명령어를 입력하여 `MyKey.pem`의 권한을 설정합니다.

            ```bash
            chmod 600 MyKey.pem
            ```

            
        * ssh 명령어로 인스턴스에 접속합니다.

            ```PowerShell
            ssh -i MyKey.pem ubuntu@복사한 linux-server-basic 플로팅 IP 주소
            ```

        * "Are you sure you want to continue connecting (yes/no/[fingerprint])?" 문구가 나오면 `yes` 를 입력 후 엔터키를 입력합니다.
        * lsb\_release 명령어로 현재 접속한 Linux 버전을 확인합니다.

            ```bash
            lsb_release -a
            ```


## 단계 2. 웹 서버 설치 및 구동하기

1. 인스턴스에 원격 접속한 상태에서 아래 명령어를 입력하여 Nginx 웹 서버를 설치합니다.

    ```bash
    sudo apt update -y
    sudo apt install nginx -y
    ```
2. 설치가 완료되면 아래 명령어로 웹 서버가 구동되는지 확인합니다.

    ```bash
    curl localhost
    ```
    
![4 네트워크설정과 인스턴스 생성_작업5 스크린샷_rv1](https://github.com/user-attachments/assets/3bc07c8b-d6e2-431d-aad4-da3bfa6562e4)

# 참고 자료

* [리전 가이드](https://docs.nhncloud.com/ko/nhncloud/ko/region-guide/)
* [Compute Instance](https://docs.nhncloud.com/ko/Compute/Instance/ko/overview/)
* [System Image](https://en.wikipedia.org/wiki/System_image)
* [VPC](https://docs.nhncloud.com/ko/Network/VPC/ko/overview/)
* [Subnet](https://docs.nhncloud.com/ko/Network/VPC/ko/console-guide/#_4)
* [Floating IP](https://www.nhncloud.com/kr/service/network/floating-ip)
* [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing)
* [키페어(Key-pair)](https://docs.nhncloud.com/ko/Compute/Instance/ko/overview/#key-pair)
* [Public-key 암호화](https://en.wikipedia.org/wiki/Public-key_cryptography)
* [SSH](https://en.wikipedia.org/wiki/Secure_Shell)
* [VM](https://en.wikipedia.org/wiki/Virtual_machine)
* [Internet Gateway](https://docs.nhncloud.com/ko/Network/Internet%20Gateway/ko/overview/)
* [Nginx](https://en.wikipedia.org/wiki/Nginx)
* [웹서버](https://en.wikipedia.org/wiki/Web_server)
* [Linux](https://en.wikipedia.org/wiki/Linux)
* [Network Interface](https://docs.nhncloud.com/ko/Network/Network%20Interface/ko/overview/)

# 이전 단계

* [03-IAM 계정과 거버넌스 설정](dooray://1387695619080878080/pages/3977493217025617647 "publish")


# 다음 단계

* [05-보안 설정](dooray://1387695619080878080/pages/3959371258218176884 "publish")
