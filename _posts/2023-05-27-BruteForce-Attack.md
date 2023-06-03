## Bruteforce 공격(무차별 대입 공격)

### 개요

이번 1주차 Writeup 에서는 Bruteforce 공격(무차별 대입 공격)에 대한 공부 및 실습을 진행해보았다. 아래 설명하는 모든 공부 및 실습은 VirtualBox - Kali Linux 가상머신에서 DVWA 취약한 웹 환경을 구성하여 이루어졌다.

### 목차

이번 Writeup 에서 진행하는Bruteforce 공격과 관련된 공부는 다음과 같다.
1. 취약점 정보/설명 - Bruteforce 공격에 대한 연구
2. 개념 증명 실습 - Burpsuite Intruder 툴을 활용하여 DVWA Security Level(Low, Midium, High)로 구성된 실습 진행
3. 대응방안 공부 - Bruteforce 공격에 대한 대응 방안
4. 레퍼런스

### 취약점 정보



### 취약점 설명

Bruteforce 공격이란?

Bruteforce 공격은 무차별 대입 공격이라고도 하며, 사용자의 비밀번호, PIN번호, 암호화키 등을 무작위로 계속 대입함으로써 해킹을 시도하는 공격 방식이다. 웹 서버/웹 어플리케이션을 대상으로 하는 경우 기본 유저 이름(default username)을 알아낸 뒤 로그인이 가능할 때 까지 비밀번호를 무작위로 대입하는 방식으로 많이 이루어진다. 공격자는 Bruteforce 공격을 통해 사용자의 자격 증명을 찾아낸 뒤 해당 사용자로 시스템에 접근하여 계정을 장악하거나 추가 권한 상승을 시도할 수 있다. 찾아낸 자격 증명이 관리자인 경우에는 웹 어플리케이션 전체 장악까지 이루어져 큰 피해를 입을 수 있다.

다양한 종류의 bruteforce가 존재하지만 여기에서는 사전 공격을 통한 bruteforce 실습을 진행해보았다.

### 개념 증명

#### * Low Level

##### Burpsuite Intruder

먼저 공격자는 admin 계정을 알고 있다고 가정한 뒤, 비밀번호에 대한 사전 대입 공격을 진행한다. Burpsuite을 통해 Proxy 탭에서 사용자가 인증 할 때 HTTP 요청을 가로챈 뒤, 이를 Intruder 탭으로 넘겨보았다. 이를 위해 password= 파라미터에 $ 기호를 넣어 해당 파라미터만 사전 대입 공격을 할 수 있도록 하였다.

![이미지](/assets/3_intruder.png)

먼저 `gedit /usr/share/john/password.lst` 입력하여 칼리리눅스에 있는 딕셔너리를 확인할 수 있었다.

![이미지](/assets/5_gedit.png)

password.lst 파일 안에는 1996년 부터 2011년 까지 사용자들이 자주 사용하는 패스워드들이 담겨 있다. 현재는 온라인에 다양한 유추 가능한 패스워드들이 존재하니 그 것을 사용해도 좋다. 성공적인 실습을 위해 `password` 가 있는지 확인 후에 Start Attack 을 눌러 공격을 수행하였다.

![이미지](/assets/6_john파일.png)

admin 계정에 유효한 패스워를 통해 로그인에 성공할 경우 HTTP 응답의 크기가 4704가 되는 것을 확인할 수 있었다. Payloads를 보니 응답값으로 'Welcome to the password protected area admin' 이라는 문구를 출력하는 것을 볼 수 있었다.

* bruteforce 공격에 실패하였을 경우
![이미지](/assets/wrong_pass.png)

* bruteforce 공격에 성공하였을 경우
![이미지](/assets/pass.png)

공격자가 알아낸 패스워드로 로그인을 시도하였더니, 성공적으로 사용자의 계정을 탈취하여 로그인에 성공하는 것 까지 확인할 수 있었다. 

![이미지](/assets/2.png)

* 소스코드(Low)
```c
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

위 소스코드에서 bruteforce 공격을 감지하거나 대응할 수 있는 코드가 확인 되지 않았다. bruteforce 공격에 대응할 시큐어코딩이 필요해 보인다. 

#### * Midium Level

#### * High Level

### 취약점 원인

###### BruteForce 공격에 취약한 함수

### 대응 방안

1. 길고 복잡하며 암호화된 비밀번호를 사용한다(256비트 암호화가 이상적)

2. 비밀번호 해시에 솔트(salt)를 넣는다.

3. 비밀번호 복잡성과 여러 계정에서의 비밀번호 재사용에 관한 정책을 설정한다.

4. 비밀번호 인증에 속도 제한을 적용한다.

5. 캡차(captcah)를 활성화한다.

### 레퍼런스
- Bruteforce 공격에 대한 설명 : https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup#undefined-5
