# 개요

이번 내용에서는 NHN Cloud 환경에서 보안을 설정하고 관리하는 기본 개념과 주요 기능을 단계별로 안내하여, 안전하고 신뢰할 수 있는 클라우드 환경을 구축하도록 돕습니다. NHN Cloud는 사용자의 데이터를 안전하게 보호하고, 클라우드 자원을 효율적으로 관리할 수 있는 다양한 보안 기능을 제공합니다.

# 주요 내용

* **네트워크 보안 설정**
    * VPC (Virtual Private Cloud) 및 Subnet 구성으로 네트워크를 분리하고 보호합니다.
    * 보안 그룹(Security Group)과 ACL(Access Control List)을 설정하여 허가된 트래픽만 허용하는 방식을 학습합니다.
* **데이터 보안**
    * NHN Cloud에서 제공하는 데이터 암호화 방식 및 스토리지 보안 설정 방법을 다룹니다.
    * 백업 및 복구 전략을 통해 데이터 손실에 대비하는 방법을 소개합니다.

# 설정 및 요건

NHN Cloud를 시작하기 위해서는 다음 사항을 준비해야 합니다.

* **표준 인터넷 브라우저**
    * Google Chrome, Microsoft Edge, Firefox, Safari 등의 최신 버전 브라우저가 설치되어 있어야 합니다.
    * 브라우저 설정에서 JavaScript 및 쿠키가 활성화되어 있어야 합니다.
* **인터넷 연결 환경**
    * 안정적인 인터넷 연결이 필요하며, 권장 대역폭은 최소 5Mbps 이상입니다.
    * HTTPS를 통한 안전한 통신이 가능해야 합니다.
* [04-네트워크 설정과 인스턴스 생성](dooray://1387695619080878080/pages/3959371119803939102 "publish") 실습 완료
<br>

<br>
본 가이드는 [04-네트워크 설정과 인스턴스 생성](dooray://1387695619080878080/pages/3959371119803939102 "publish")  이후 단계부터 시작됩니다.

# 작업

## 작업 1. 웹 서버 인스턴스에 외부 접근을 허용하기 위한 보안 규칙 적용

1. 콘솔창 왼쪽 메뉴 중 **Network - Floating IP** 를 클릭합니다.
2. 플로팅 IP 리소스 목록 중 연결된 장치가 `linux-server-basic` 인 IP 주소를 **복사** 후 **기록**합니다.
3. 웹 브라우저에서 새 창을 열어서 `http://복사한 linux-server-basic 플로팅 IP 주소` 를 입력하여 접속을 확인합니다.
    * 정보: 접속 결과
        * <span style="color:#e11d21;">**해당 웹 서버에 접속하면 웹 페이지가 보이지 않습니다.**</span> 이는 해당 웹 서버 Network Interface에 보안 그룹을 설정하지 않아 모든 통신이 차단된 상태이기 때문입니다.
4. 웹 브라우저에서 콘솔창으로 돌아온 후 왼쪽 메뉴에서 **Network - Security Groups** 를 클릭합니다.
5. **Security Groups 화면**에서 <strong>[ + 보안 그룹 생성 ]</strong> 버튼을 클릭합니다.
6. **보안 그룹 생성 창**에서 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * 이름: `MySG-HTTP`
    * 보안 규칙 추가에  **[ + ]** 버튼을 클릭하여 다음 설정의 보안 규칙을 추가
        * 방향: `수신`
        * IP 프로토콜: `HTTP`
        * Ether: `IPv4`
        * 원격: `CIDR - 0.0.0.0/0`
7. 성공 창에서 **[확인]** 버튼을 클릭합니다.
8. 콘솔창 왼쪽 메뉴에서 **Compute - Instance** 를 클릭합니다.
9. <span style="color:rgb(51, 51, 51);">**Instance** 화면에서 `linux-server-basic` 인스턴스를 클릭하여 선택합니다.</span>
10. 하단 분할창에서 **[네트워크]** 탭을 클릭합니다.
11. 인스턴스 생성 시 함께 **자동으로 생성된 네트워크 인터페이스 리소스(네트워크 인터페이스 ID는 임의의 ID로 지정됨)** 를 클릭하여 선택합니다.
12. 네트워크 인터페이스 목록 바로 위에 **[보안 그룹 변경]** 버튼을 클릭합니다.
13. **보안 그룹 변경** 창에서 보안 그룹 선택 목록 중 현 상태에서 `MySG-HTTP` 를 체크하여 추가 후 **[확인]** 버튼을 클릭합니다.
14. 웹 브라우저에서 새 창을 열어서 `http://복사한 linux-server-basic 플로팅 IP 주소` 를 입력하여 접속을 확인합니다.
15. 웹 페이지가 정상적으로 출력되는 것을 확인합니다.

<br>
## 작업 2. 웹 서버 인스턴스에 허용된 IP만 SSH, ICMP(Ping 등) 통신 허용하기

1. 콘솔창 왼쪽 메뉴 중 **Network - Security Groups** 를 클릭합니다.
2. **Security Groups 화면**에서 `MySG-SSH` 를 클릭하여 선택합니다.
3. 하단 분할창 **보안 규칙** 탭에서 포트 범위가 **22 (SSH)** 인 규칙을 선택 후 오른쪽에 **[변경]** 버튼을 클릭합니다.
4. **보안 규칙 변경** 창에서 원격 항목에 `CIDR` 드롭박스를 클릭 후 `내 IP`를 선택 후 **[확인]** 버튼을 클릭합니다.
5. 성공 창에서 **[확인]** 버튼을 클릭합니다.
6. **[+ 보안 규칙 생성]** 버튼을 클릭합니다.
7. <strong>보안 규칙 생성</strong> 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * 방향: `수신`
    * IP 프로토콜: `ALL ICMP`
    * Ether: `IPv4`
    * 원격: `내 IP`
8. 성공 창에서 **[확인]** 버튼을 클릭합니다.
9. 새로운 **터미널** 또는 **PowerShell** 을 실행합니다.
10. 아래 명령어로 통신 테스트를 진행합니다.

    ```bash
    ping (linux-server-basic 플로팅 IP 주소)
    ```

    <span style="color:#0052cc;">**Ping 통신이 허용**</span>된 것을 확인합니다.

<br>
## 작업 3. 특정 네트워크 대역 통신 차단을 위한 Network ACL 설정

1. 콘솔창 왼쪽 메뉴 중 **Network - Network ACL** 을 클릭합니다.
2. **Network ACL > 관리 화면**에서 <strong>[+ Network ACL 생성]</strong> 버튼을 클릭합니다.
3. **Network ACl 생성** 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * 이름: `MyACL`
4. 성공 창에서 **[확인]** 버튼을 클릭합니다.
5. 생성된 `MyACL` 을 클릭하여 선택합니다.
6. 하단 분할창에서 **ACL Rule** 탭을 클릭합니다.
7. MyACL의 ACL Rule 목록에서 설명에 **"default allow rule"** 이 표시된 **101** 순서의 ACL Rule을 선택 후 **[ACL Rule 삭제]** 버튼을 클릭합니다.
8. 성공 창에서 **[확인]** 버튼을 클릭합니다.
9. 화면 상단에 **Network ACL 탭 중 바인딩** 탭을 클릭합니다.
10. **[ + ACL 바인딩 생성 ]** 버튼을 클릭합니다.
11. ACL 바인딩 생성 창에서 아래 정보를 설정 후 [확인] 버튼을 클릭합니다.
    * Network ACL: `MyACL`
    * VPC: `MyVPC`
12. 성공 창에서 **[확인]** 버튼을 클릭합니다.
13. **터미널** 또는 **PowerShell** 에서 아래 명령어를 실행하여 통신 테스트를 진행합니다.

    ```bash
    ping (linux-server-basic 플로팅 IP 주소)
    ```

    <span style="color:#e11d21;">**Ping 통신이 차단**</span>된 것을 확인합니다.
14. 웹 브라우저에서 새 창을 열어서 `http://복사한 linux-server-basic 플로팅 IP 주소` 를 입력하여 접속을 확인합니다.
    <span style="color:#e11d21;">**http 통신이 차단**</span>된 것을 확인합니다.

<br>
## 작업 4. Network ACL 추가 Rule 적용

1. **[ + ACL Rule 생성 ]** 버튼을 클릭합니다.
2. **ACL Rule 생성** 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * IP 프로토콜: `ICMP`
    * 출발지 CIDR: `0.0.0.0/0`
    * 목적지 CIDR: `linux-server-basic 플로팅 IP 주소/32`
    * 순서: `110`
    * 적용 방법: `차단`
3. 성공 창에서 **[확인]** 버튼을 클릭합니다.
4. **[ + ACL Rule 생성 ]** 버튼을 클릭합니다.
5. **ACL Rule 생성** 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * IP 프로토콜: `ICMP`
    * 출발지 CIDR: `작업하는 컴퓨터의 IP 주소/32`
        * 팁: 작업하는 컴퓨터의 IP 주소 확인 방법
            * **터미널** 또는 **PowerShell** 을 실행 후 `curl ifconfig.me` 를 실행하면 **본인이 작업하는 컴퓨터의 IP 주소**를 확인할 수 있습니다.
    * 목적지 CIDR: `linux-server-basic 플로팅 IP 주소/32`
    * 순서: `109`
    * 적용 방법: `허용`
6. **ACL Rule 생성** 창에서 아래 정보를 설정 후 **[확인]** 버튼을 클릭합니다.
    * IP 프로토콜: `전체적용`
    * 출발지 CIDR: `0.0.0.0/0`
    * 목적지 CIDR: `0.0.0.0/0`
    * 순서: `32764`
    * 적용 방법: `허용`
7. **터미널** 또는 **PowerShell** 에서 아래 명령어를 실행하여 통신 테스트를 진행합니다.

    ```bash
    ping (linux-server-basic 플로팅 IP 주소)
    ```

    <span style="color:#0052cc;">**작업하는 컴퓨터의 IP**</span><span style="">에서만 </span><span style="color:#0052cc;">**Ping 통신이 허용**</span>된 것을 확인합니다.
8. 웹 브라우저에서 새 창을 열어서 `http://복사한 linux-server-basic 플로팅 IP 주소` 를 입력하여 접속을 확인합니다.
    <span style="color:#0052cc;">**모든 IP**</span><span style="">에서 </span><span style="color:#0052cc;">**http 통신이 허용**</span>된 것을 확인합니다.
<br>

<br>
# 참고 사이트

* [보안 그룹](https://docs.nhncloud.com/ko/Network/Security%20Groups/ko/overview/)
* [Network ACL](https://docs.nhncloud.com/ko/Network/Network%20ACL/ko/overview/)
* [Network Port](https://en.wikipedia.org/wiki/Port_(computer_networking))
* [ACL](https://en.wikipedia.org/wiki/Access-control_list)
* [Whitelist](https://en.wikipedia.org/wiki/Whitelist)
* [Blacklist](https://en.wikipedia.org/wiki/Blacklist_(computing))
* [TCP](https://en.wikipedia.org/wiki/TCP)
* [ICMP(Internet Control Message Protocol)](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol)
* [ping](https://en.wikipedia.org/wiki/Ping_(networking_utility))

# 이전 단계

* [04-네트워크 설정과 인스턴스 생성](dooray://1387695619080878080/pages/3959371119803939102 "publish")
<br>

# 다음 단계

* [06-데이터베이스 생성 및 연결](dooray://1387695619080878080/pages/3966825684645477138 "publish")

<br>
<br>
<br>
<br>
<br>
<br>
