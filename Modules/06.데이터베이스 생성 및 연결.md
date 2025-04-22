# 개요

이번 학습 모듈에서는 NHN Cloud 환경에서 데이터베이스를 생성하고 애플리케이션과 연결하는 기본적인 구성 절차를 안내합니다. NHN Cloud에서는 안정적이고 확장 가능한 **Database 서비스**를 제공하여 사용자가 쉽고 효율적으로 데이터베이스를 구축하고 운영할 수 있도록 지원합니다.

# 학습 목표

이번 학습 모듈에서 배울 내용은 다음과 같습니다.

* **Database 서비스 활용**
    * 데이터베이스 인스턴스 생성 및 초기 설정
    * 데이터베이스 접속 및 간단한 데이터 작업

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
    * 본 가이드는 [05-보안 설정](dooray://1387695619080878080/pages/3959371258218176884 "publish") 이후 단계부터 시작됩니다.

# 데이터베이스 생성 및 데이터 조회

## 단계 1. MySQL 데이터베이스 인스턴스 만들기

1. NHN Cloud 콘솔 상단 메뉴에서 실습에 사용할 조직(`MyORG`), 프로젝트(`MyPRJ`), 그리고 `한국(평촌) 리전`을 선택합니다.
2. 콘솔 창 왼쪽 메뉴 중 **Database - MySQL Instance**를 클릭합니다.
3. **인스턴스 생성**을 클릭합니다.
4. **인스턴스 생성** 창에서 아래 정보를 설정 후 **인스턴스 생성**을 클릭합니다.
    * 인스턴스 템플릿
        * 사용: `사용 안 함`
    * 이미지
        * 공용 이미지 > 이미지 이름: `Ubuntu Server 20.04 LTS` 클릭
    * 인스턴스 정보
        * 가용성 영역: `임의의 가용성 영역`
        * 인스턴스 이름: `mysql-db-basic`
        * 인스턴스 타입: **인스턴스 타입 선택** > 인스턴스 타입 이름에서 `t2.c1m1` 클릭한 뒤 **선택** 클릭
        * 인스턴스 수: `1`
        * 키페어 > `MyKey`
            * 키페어를 추가로 생성하여 사용하려면 [키페어 사용자 가이드](https://docs.nhncloud.com/ko/Compute/Instance/ko/console-guide/#_21)를 참고하시기 바랍니다.
    * 루트 블록 스토리지
        * 블록 스토리지 타입: `HDD`
        * 블록 스토리지 크기(GB): `20` GB
    * 네트워크 설정: `네트워크 인터페이스 생성` 선택
    * 네트워크
        * 사용 가능한 서브넷 항목에서 `MySubnet (192.168.0.0/24)` 리소스를 클릭하여 선택된 서브넷으로 사용
    * 플로팅 IP: **설정 변경** 클릭
        * 사용: `사용 안함 (Default)`
    * 보안 그룹
        * **보안 그룹 생성** 클릭
        * **보안 그룹 생성** 창에서 아래 정보를 설정 후 **확인**을 클릭합니다.
            * 이름: `MySG-DB`
            * 보안 규칙 추가에  **+**을 클릭한 뒤 다음 설정의 보안 규칙을 추가
                * 방향: `수신`
                * IP 프로토콜: `My SQL`
                    * 해당 IP 프로토콜을 선택하면 자동으로 포트 정보가 입력됩니다.        
            * Ether: `IPv4`
            * 원격: `CIDR - 192.168.0.0/24`
        * 성공 창에서 **확인**을 클릭합니다.
        * **보안 그룹 선택** 항목에서 위에 생성한 `MySG-DB`를 선택합니다.


!!! tip "알아두기"
    * 사용자 정의 TCP
        * IP 프로토콜을 "사용자 정의 TCP"로 선택하면 포트값을 사용자가 직접 입력할 수 있습니다. **"1"**로 표기된 입력칸에 `3306` 을 입력하시기 바랍니다.

    * 추가 블록 스토리지: 사용 안 함 (기본)
    * 사용자 스크립트

        ```bash
        #!/bin/bash
        
        # Variable setup
        DB_NAME="employees"
        USER_NAME="nhncloud"
        USER_PASSWORD="nhnpassword"
        DB_URL="https://github.com/datacharmer/test_db.git"
        
        # 1. Clone the test_db repository
        echo "Cloning test_db repository..."
        cd ~
        git clone $DB_URL
        
        # 2. Load employees.sql file into MySQL
        echo "Loading employees.sql into MySQL..."
        cd ~/test_db
        sudo mysql -u root < employees.sql
        
        # 3. Configure MySQL user and permissions
        echo "Configuring MySQL user and permissions..."
        sudo mysql -u root <<EOF
        USE $DB_NAME;
        
        # Create user (for localhost and 192.168.0.% range)
        CREATE USER '$USER_NAME'@'localhost' IDENTIFIED BY '$USER_PASSWORD';
        CREATE USER '$USER_NAME'@'192.168.0.%' IDENTIFIED BY '$USER_PASSWORD';
        
        # Grant privileges
        GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$USER_NAME'@'localhost';
        GRANT ALL PRIVILEGES ON $DB_NAME.* TO '$USER_NAME'@'192.168.0.%';
        
        # Apply privileges
        FLUSH PRIVILEGES;
        EOF
        
        echo "MySQL user '$USER_NAME' created and configured successfully!"
        ```

    * 삭제 보호: 사용 안 함 (기본)
4. 인스턴스 생성 정보 창에서 **인스턴스 생성**을 클릭합니다.
5. 인스턴스 생성 작업이 진행됩니다. 해당 인스턴스는 약 1분 내외로 생성이 완료됩니다.
6. 인스턴스 생성 후 `mysql-db-basic`인 IP 주소를 **복사** 후 **기록**합니다.

## 단계 2. 리눅스 인스턴스에서 데이터베이스 접속 테스트 하기

> 모듈 3에서 생성한 리눅스 인스턴스 `linux-server-basic`을 사용해 데이터베이스에 접속합니다. linux-server-basic 인스턴스 생성 및 접속 방법은 [04-네트워크 설정과 인스턴스 생성](dooray://1387695619080878080/pages/3959371119803939102 "publish")을 참고하시기 바랍니다.

1. 새로운 **터미널** 또는 **PowerShell**을 실행합니다.
2. 아래 명령어로 `linux-server-basic`에 원격 접속합니다.

    ```PowerShell
    cd /(MyKey.pem 파일이 있는 폴더명)
    ```

    ```PowerShell
    ssh -i MyKey.pem ubuntu@(linux-server-basic 인스턴스의 Floating IP 주소)
    ```

3. 원격 접속 환경에서 아래 명령어를 실행하여 **MySQL-Client** 툴을 설치합니다.

    ```bash
    sudo apt update 
    sudo apt install mysql-client -y
    ```

4. 아래 명령어를 실행하여 데이터베이스의 결과값을 확인합니다.

    ```bash
    export MYSQL_PWD=nhnpassword
    ```

    ```bash
    mysql --host=(mysql-db-basic 인스턴스의 가상 IP 주소) --user=nhncloud -e "SELECT * FROM employees.employees LIMIT 30;"
    ```

**데이터베이스의 결과값이 조회**되는 것을 확인합니다.

{pic}

# 참고 자료

* [MySQL](https://en.wikipedia.org/wiki/MySQL)
* [Database](https://en.wikipedia.org/wiki/Database)
* [SQL](https://en.wikipedia.org/wiki/SQL)
* [RDS for MySQL](https://docs.nhncloud.com/ko/Database/RDS%20for%20MySQL/ko/overview/)

# 이전 단계

* [05-보안 설정](dooray://1387695619080878080/pages/3959371258218176884 "publish")

# 다음 단계

* [07-스토리지 생성 및 설정](dooray://1387695619080878080/pages/3996521605058381185 "publish")
