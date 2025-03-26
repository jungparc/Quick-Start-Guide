# 개요

이번 학습 모듈에서는 NHN Cloud 콘솔을 통해 스토리지 서비스를 활성화하고 사용하는 방법을 안내합니다. NHN Cloud의 **스토리지 서비스**는 데이터 저장과 관리에 필요한 안정적이고 확장 가능한 솔루션을 제공합니다.

# 학습 목표

이번 학습 모듈에서 배울 내용은 다음과 같습니다.

* **Storage 서비스 활용**
    * Object Storage와 Block Storage의 차이점과 사용 사례 비교
    * Block Storage 볼륨 생성 및 인스턴스와 연결
    * Object Storage 컨테이너 생성 및 객체 업로드와 다운로드

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
    * NHN Cloud 홈페이지에 로그인 해야 합니다.

!!! tip "알아두기"
    * 본 가이드는 [06-데이터베이스 생성 및 연결](dooray://1387695619080878080/pages/3966825684645477138 "publish") 이후 단계부터 시작됩니다.

# 블록 스토리지 생성 및 데이터 조회

> 블록 스토리지를 생성해 리눅스 인스턴스에 연결한 뒤 블록 스토리지의 데이터를 조회합니다.

## 단계 1. 블록 스토리지 생성 후 리눅스 인스턴스에 연결하기

> 모듈 3에서 생성한 리눅스 인스턴스 `linux-server-basic`에 블록 스토리지 리소스를 연결합니다.

1. 콘솔창 왼쪽 메뉴 중 **Storage - Block Storage**를 클릭합니다.
2. **Block Storage > 관리 화면**에서 블록 스토리지 목록 중 연결 정보가 **linux-server-basic의 /dev/vda** 인 블록스토리지의 가용성 영역을 확인합니다.
3. **+ 블록 스토리지 생성**을 클릭합니다.
4. **블록 스토리지 생성** 창에서 아래 정보를 설정 후 **확인**을 클릭합니다.
    * 블록 스토리지 이름: `MyBS`
    * 블록 스토리지 타입: `HDD`
    * 블록 스토리지 크기(GB): `10 GB`
    * 가용성 영역: 블록 스토리지 목록에서 연결정보가 `linux-server-basic의 /dev/vda`인 블록 스토리지와 **동일한 가용성 영역**
5. 성공 창에서 **확인**을 클릭합니다.
6. `MyBS`를 선택 후 위에 **연결 관리**를 클릭합니다.
7. **블록 스토리지 연결 관리** 창에서 아래 정보를 설정 후 **연결**을 클릭합니다.
    * 인스턴스에 연결: `linux-server-basic`
8. 성공 창에서 **확인**을 클릭합니다.

## 단계 2. 추가 블록 스토리지 파티션 설정, 포맷, 마운트하기

1. 리눅스 인스턴스 `linux-server-basic`에 원격 접속한 상태에서 아래 명령어를 실행해 단계 1에서 생성한 블록 스토리지 `MyBS`에 파티션을 생성한 뒤 포맷과 마운트를 실행합니다.

    ```bash
    echo -e "n\np\n1\n\n\nw" | sudo fdisk /dev/vdb
    ```

<details><summary>도메인 이름 가이드</summary>
<p>
    * fdisk 명령어 설명
        * **`n`**: 새 파티션 생성 (옵션: `n`)
            * **`p`**: 파티션 유형 선택 (기본값: Primary)
            * **`1`**: 파티션 번호 (기본값: 1번)
            * **첫 번째 섹터**: Enter를 눌러 기본값 선택
            * **마지막 섹터**: Enter를 눌러 기본값으로 최대 크기 선택
        * **`w`**: 변경 내용을 저장하고 종료
</p>
</details>

2. 아래 명령어를 실행해 파티션을 포맷합니다.

    ```bash
    sudo mkfs -t xfs /dev/vdb1
    ```

   추가한 파티션을 /mnt/vdb 경로에 마운트 할 수 있도록 아래 명령어를 실행하여 /mnt/vdb 디렉토리를 생성합니다.

    ```
    sudo mkdir /mnt/vdb
    ```

    부팅 시 추가한 파티션을 자동 마운트하도록 아래 명령어를 실행하여 /etc/fstab 파일에 설정을 추가합니다.

    ```
    sudo sh -c 'echo "UUID=$(sudo blkid -s UUID -o value /dev/vdb1) /mnt/vdb xfs defaults,nodev,noatime,nofail 1 2" >> /etc/fstab'
    ```

    아래 명령어를 실행하여 /etc/fstab 파일에 정의된 모든 파일 시스템을 마운트하고, 모든 사용자가 /mnt/vdb에 파일 생성, 수정, 삭제 가능하도록 권한을 변경합니다.

    ```bash
    sudo mount -a
    sudo chmod 777 /mnt/vdb
    ```

3. 아래 명령어를 입력해 추가 블록 스토리지가 올바르게 설정되었는지 확인합니다.

    ```bash
    df /dev/vdb1
    ```

    **해당 Filesystem과 용량, Mount 경로가 조회**되는 것을 확인합니다.

{pic}

## 단계 3. 추가 블록 스토리지에 자료 생성하기

1. `linux-server-basic`에 원격 접속한 상태에서 아래 명령어를 실행하여 블록 스토리지를 생성합니다.

    ```bash
    export MYSQL_PWD=nhnpassword
    ```

    생성한 블록 스토리지에 `mysql-db-basic` 데이터 조회 자료를 생성합니다.

    ```bash
    mysql --host=(mysql-db-basic 인스턴스의 가상 IP 주소) --user=nhncloud -e "SELECT * FROM employees.employees LIMIT 30;" -B --batch > /mnt/vdb/employees.csv
    ```

2. 아래 명령어로 생성한 데이터(<span style="color:rgb(0, 82, 204);"><strong>csv 파일)</strong></span>를 확인합니다.

    ```bash
    cat /mnt/vdb/employees.csv
    ```
{pic}

# 오브젝트 스토리지 생성 및 데이터 조회

> 오브젝트 스토리지 서비스를 활성화해 컨테이너를 생성한 뒤 객체를 업로드합니다. 해당 객체를 리눅스 인스턴스에 저장한 뒤 접속해 데이터가 출력되는 지 확인합니다.

## 단계 1. 오브젝트 스토리지 서비스 활성화 및 컨테이너 생성하기

1. `MyPRJ` 프로젝트 오른쪽에 위치한 **"서비스 선택"** 탭을 클릭합니다.
2. **서비스 선택** 클릭 후 나오는 화면에서 **모든 서비스 - Storage - Object Storage**를 클릭합니다.
3. **서비스 활성화** 창에서 **확인**을 클릭합니다.
4. 서비스가 올바르게 활성화되면 Object Storage 화면으로 이동됩니다. 만약 화면이 이동하지 않을 경우 콘솔 창 왼쪽 메뉴 중 **Storage - Object Storage**를 클릭합니다.
5. **+ 컨테이너 생성**을 클릭합니다.
6. **컨테이너 생성** 창에서 아래 정보를 입력 후 **확인**을 클릭합니다.
    * 이름: `myobs`
    * 접근 정책: `PUBLIC`
    * 스토리지 클래스: `Standard`
    * 객체 잠금 설정: `사용 안 함`
    * 암호화 설정: `사용 안 함`
7. 성공 창에서 **확인**을 클릭합니다.

## 단계 2. 오브젝트 스토리지를 이용해 리눅스 인스턴스 웹 소스 변경하기

1. 사용자 작업 환경에서 새로운 **터미널** 또는 **PowerShell**을 실행합니다.
2. 아래 명령어로 web-sample 디렉토리를 생성하고 index.html 파일을 저장합니다.

    ```PowerShell
    mkdir /web-sample
    ```

    ```PowerShell
    echo '<!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>::: Welcome to NHN Cloud :::</title>
        <script>
            let countdownTimer;
    
            function fetchServerIP() {
                fetch("/server-info")
                    .then(response => response.json())
                    .then(data => {
                        const serverIP = data.serverIP;
                        document.getElementById("server-ip").innerText = "Server IP Address: " + serverIP;
                    })
                    .catch(error => {
                        document.getElementById("server-ip").innerText = "Unable to fetch server IP address";
                    });
            }
    
            function startStressTest() {
                const button = document.getElementById("start-btn");
                button.innerText = "Starting...";
                let timeLeft = 120;
                const countdownDisplay = document.getElementById("countdown");
                countdownDisplay.innerText = "Time left: " + timeLeft + " seconds";
    
                countdownTimer = setInterval(() => {
                    timeLeft--;
                    countdownDisplay.innerText = "Time left: " + timeLeft + " seconds";
    
                    if (timeLeft <= 0) {
                        clearInterval(countdownTimer);
                        countdownDisplay.innerText = "Stress test complete!";
                    }
                }, 1000);
    
                fetch("/start-stress")
                    .then(response => response.text())
                    .then(data => {
                        console.log(data);
                    })
                    .catch(error => {
                        console.error("Error starting stress test:", error);
                    });
            }
    
            window.onload = fetchServerIP;
        </script>
    </head>
    <body>
        <h1>Welcome to NHN Cloud Web Server</h1>
        <p id="server-ip">Fetching server IP address...</p>
        <button id="start-btn" onclick="startStressTest()">Start Stress Test</button>
        <p id="countdown">Time left: 120 seconds</p>
    </body>
    </html>' > /web-sample/index.html
    ```

3. 콘솔창 왼쪽 메뉴 중 **Storage - Object Storage**를 클릭합니다.
4. Object Storage 화면에서 `myobs`를 클릭하여 컨테이너 상세 페이지 화면으로 이동합니다.
5. `myobs` 화면에서 **객체 업로드**를 클릭합니다.
6. **객체 업로드** 창에서 **파일 선택**을 클릭 후 **/web-sample**에 위치한 `index.html` 파일을 선택 후 **확인**을 클릭합니다.
7. **업로드 상태 정보** 창에서 `index.html` 파일 업로드 상태가 **성공**인 것을 확인 후 **확인**을 클릭합니다.
8. 업로드한 `index.html` 객체 오른쪽에 **Public URL** 항목에 있는 **URL 복사**를 클릭해 **index.html 파일의 Public URL 주소**를 복사합니다.
9. `linux-server-basic`에 원격 접속을 합니다.

!!! tip "알아두기"
    * `linux-server-basic`에 원격 접속 방법
        * 해당 원격 접속 방법은 [04-네트워크 설정과 인스턴스 생성](dooray://1387695619080878080/pages/3959371119803939102 "publish") - **작업 4. SSH 원격 접속** 방법을 참고바랍니다.

10. `linux-server-basic` 원격 접속 후 아래 명령어를 실행해 index.html을 다운로드한 후 저장합니다.

    ```bash
    sudo curl -o /var/www/html/index.html (myobs에 업로드한 index.html 파일의 Public URL)
    ```

!!! tip "알아두기"
    * Nginx의 기본 웹 문서 경로
        * Ubuntu에서 설치한 Nginx의 기본 웹 문서 경로는 `/var/www/html`입니다. `http://복사한 linux-server-basic 플로팅 IP 주소`에 접속 시 출력되는 웹 페이지 문서는 `/var/www/html/index.html` 입니다.

12. 아래 명령어를 실행하여 추가 서비스 구성을 설정합니다.

    ```bash
    curl -s https://kr2-api-object-storage.nhncloudservice.com/v1/AUTH_cd41d4b57c1346b99641cc4092f2842d/onboarding/service-setting.sh | sed 's/\r$//' > /home/ubuntu/service-setting.sh
    chmod +x /home/ubuntu/service-setting.sh
    /home/ubuntu/service-setting.sh
    ```

13. 웹 브라우저에서 새 창을 열어서 `http://복사한 linux-server-basic 플로팅 IP 주소`를 입력하여 변경된 웹 페이지를 확인합니다.
    ![Inline-image-2025-01-06 09.16.13.635.png](/wikis/3789776849324644692/files/3996525313124100147)

!!! tip "알아두기"
    * 변경된 웹 페이지가 보이지 않을 경우
        * 웹 페이지가 변경되었음에도 불구하고 이전 페이지가 표시된다면, 웹 브라우저의 캐시로 인해 발생한 문제일 수 있습니다. 이 경우, 브라우저에서 제공하는 새로고침(F5) 또는 강력 새로고침을 사용하여 변경된 페이지를 확인할 수 있습니다. 강력 새로고침은 다음 방법으로 수행할 수 있습니다.
            * **Windows**: `Ctrl + F5` 또는 `Shift + F5`
            * **Mac**: `Cmd + Shift + R`


# 참고 자료

* [Storage](https://en.wikipedia.org/wiki/Cloud_storage)
* [Block Storage](https://docs.nhncloud.com/ko/Storage/Block%20Storage/ko/overview/)
* [Object Storage](https://docs.nhncloud.com/ko/Storage/Object%20Storage/ko/Overview/)
* [HDD](https://en.wikipedia.org/wiki/Hard_disk_drive)
* [SSD](https://en.wikipedia.org/wiki/Solid-state_drive)
* [Disk encryption](https://en.wikipedia.org/wiki/Disk_encryption)
* [fdisk](https://en.wikipedia.org/wiki/Fdisk)
* [DF](https://en.wikipedia.org/wiki/Df_(Unix))
* [Mount](https://en.wikipedia.org/wiki/Mount_(computing))
* [chmod](https://en.wikipedia.org/wiki/Chmod)

# 이전 단계

* [06-데이터베이스 생성 및 연결](dooray://1387695619080878080/pages/3966825684645477138 "publish")

# 다음 단계

* [08-모니터링 설정](dooray://1387695619080878080/pages/3959371359644829313 "publish")
