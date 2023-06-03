## Bruteforce 공격(무차별 대입 공격)

### 개요

이번 1주차 Writeup 에서는 Bruteforce 공격(무차별 대입 공격)에 대한 공부 및 실습을 진행해보았다. 아래 설명하는 모든 공부 및 실습은 VirtualBox - Kali Linux 가상머신에서 DVWA 취약한 웹 환경을 구성하여 이루어졌다.

### 목차

이번 Writeup 에서 진행하는Bruteforce 공격과 관련된 공부는 다음과 같다.
1. 취약점 설명 - Bruteforce 공격에 대한 연구
2. 개념 증명 실습 - Burpsuite Intruder 툴을 활용하여 DVWA Security Level(Low, Midium, High)로 구성된 실습 진행
3. 대응방안 공부 - Bruteforce 공격에 대한 대응 방안
4. 레퍼런스

### 취약점 설명

Bruteforce 공격이란?

Bruteforce 공격은 무차별 대입 공격이라고도 하며, 사용자의 비밀번호, PIN번호, 암호화키 등을 무작위로 계속 대입함으로써 해킹을 시도하는 공격 방식이다. 웹 서버/웹 어플리케이션을 대상으로 하는 경우 기본 유저 이름(default username)을 알아낸 뒤 로그인이 가능할 때 까지 비밀번호를 무작위로 대입하는 방식으로 많이 이루어진다. 공격자는 Bruteforce 공격을 통해 사용자의 자격 증명을 찾아낸 뒤 해당 사용자로 시스템에 접근하여 계정을 장악하거나 추가 권한 상승을 시도할 수 있다. 찾아낸 자격 증명이 관리자인 경우에는 웹 어플리케이션 전체 장악까지 이루어져 큰 피해를 입을 수 있다.

다양한 종류의 bruteforce가 존재하지만 여기에서는 사전 공격을 통한 bruteforce 실습을 진행해보았다.

### 개념 증명

#### * Low Level

##### Burpsuite Intruder

먼저 공격자는 admin 계정을 알고 있다고 가정한 뒤, 비밀번호에 대한 사전 대입 공격을 진행한다. Burpsuite을 통해 Proxy 탭에서 사용자가 인증 할 때 HTTP 요청을 가로챈 뒤, 이를 Intruder 탭으로 넘겨보았다. 이를 위해 password= 파라미터에 $ 기호를 넣어 해당 파라미터만 사전 대입 공격을 할 수 있도록 하였다.

![이미지](/assets/3_intruder.png)

먼저 `gedit /usr/share/john/password.lst` 입력하여 칼리리눅스에 있는 패스워드 리스트를 확인할 수 있었다.

![이미지](/assets/5_gedit.png)

password.lst 파일 안에는 1996년 부터 2011년 까지 사용자들이 자주 사용하는 패스워드들이 담겨 있다. 현재는 온라인에 다양한 유추 가능한 패스워드들이 존재하니 그 것을 사용해도 좋다. 성공적인 실습을 위해 `password` 가 있는지 확인 후에 공격을 수행하였다.

![이미지](/assets/6.john파일.png)

admin 계정에 유효한 패스워를 통해 로그인에 성공할 경우 HTTP 응답의 크기가 4704가 되는 것을 확인할 수 있었다. Payloads를 보니 응답값으로 `'Welcome to the password protected area admin'` 이라는 문구를 출력하는 것을 볼 수 있었다.

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

위 소스코드에서 Bruteforce 공격을 감지하거나 대응할 수 있는 코드가 확인 되지 않았다. Bruteforce 공격에 대응할 시큐어코딩이 필요해 보인다. 

#### * Midium Level

![이미지](/assets/2second.png)

위와 같이 취약한 웹 환경에서 Bruteforce 공격에 시도했을 때, password가 틀린 경우에는 `'Username and/or password incorrect.'` 문구를 확인할 수 있다.

![이미지](/assets/delay.png)

Midium Level 에서는 Bruteforce 공격 응답이 느리게 나오는 것을 파악할 수 있다.

```c
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
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
        sleep( 2 );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

위 소스 코드를 분석한 결과 로그인에 실패했을 경우에는 `sleep(2);` 함수를 통해 2초간 재로그인을 지연시키고 있다. 이 같은 방법으로 bruteforce 공격 진행을 지연시킬 수 있는 시큐어코딩이 되어 있지만, 해당하는 패스워드가 리스트에 상위에 있을 경우에는 여전히 사용자 권한을 빠르게 탈취할 수 있어 안심할 수 없다.

#### * High Level

High Level에서도 로그인을 시도할 경우 몇 초 정도 지연되는 것을 확인할 수 있었다.

```c
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Sanitise username input
    $user = $_GET[ 'username' ];
    $user = stripslashes( $user );
    $user = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $user ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));

    // Sanitise password input
    $pass = $_GET[ 'password' ];
    $pass = stripslashes( $pass );
    $pass = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
    $pass = md5( $pass );

    // Check database
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
       sleep( rand( 0, 3 ) );
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

소스코드 분석 결과 이번에도 `sleep( rand( 0, 3 ) );` 함수를 사용하여 로그인을 지연시키고 있지만 2초가 아니라 0~3초까지 랜덤하게 지연시키고 있는 것을 확인하였다. 

Midium에서 본 것과 같이 지연시간이 2초로 지정되어 있다면 공격자가 2초동안 응답이 없을 경우 틀린 패스워드라는 것을 간주하여 바로 다음 요청을 진행할 수 있다는 취약점을 확인할 수 있다. 따라서 시간을 랜덤하게 설정하여 서로 다른 시간으로 응답값을 받을 수 있도록 한 것을 확인할 수 있다.

### 취약점 원인


###### BruteForce 공격에 취약한 함수

### 대응 방안

1. 패스워드 복잡성과 여러 계정에서의 비밀번호 재사용에 관한 정책을 설정한다.
- 패스워드를 길고 복잡하게 만들도록 규칙을 강제하여 공격자가 패스워드를 알아낼 수 없도록 해야한다.
  (사용자 패스워드 설정 시 영문(대문자, 소문자) 숫자, 특수문자가 혼합된 패스워드로 설정하는 방법을 사용한다)

4. 자동화방지(re-Captcha)를 활성화한다.
- Bruteforce 공격의 자동화되고 반복적인 로그인 시도 자체를 불가능하게 하기위해 로그인에 반복적으로 실패할 경우에는 아래와 같은 자동화 방지 캡차를 사용한다.

![이미지](/assets/captcha.png)

5. 잘못된 로그인 시도가 지속될 경우 제한(시간, 잠금 등)을 설정한다.
- Bruteforce 공격은 임의의 문자열을 무차별 대입해보는 공격이기 때문에 엄청나게 많은 로그인 실패를 반복하게 된다. 반복적인 로그인 시도를 방지할 수 있도록 로그인 시도 횟수를 제한 할 수 있다.

### 레퍼런스
- Bruteforce 공격에 대한 설명 : [https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup#undefined-5](https://security.grootboan.com/follow-along/undefined/0-dvwa/reference-writeup#undefined-5)
- Bruteforce 공격에 대한 설명 : [https://www.kaspersky.com/resource-center/definitions/brute-force-attack](https://www.kaspersky.com/resource-center/definitions/brute-force-attack)
- reCaptcha 에 대한 설명 : [https://www.cloudflare.com/ko-kr/learning/bots/how-captchas-work/](https://www.cloudflare.com/ko-kr/learning/bots/how-captchas-work/)
