## Command Injection 공격 (명령어 주입 공격)

### 개요

이번 2주차 Writeup 에서는 Command Injection 공격(명령어 주입 공격)에 대한 공부 및 실습을 진행해보았다. 아래 설명하는 모든 공부 및 실습은 VirtualBox - Kali Linux 가상머신에서 DVWA 취약한 웹 환경을 구성하여 이루어졌다.

### 목차

이번 Writeup 에서 진행하는 Command Injection과 관련된 공부는 다음과 같다.
1. 취약점 정보/설명 - Command Inejction 공격에 대한 연구
2. 개념 증명 실습 - DVWA Security Level(Low, Midium, High)로 구성된 실습 진행
3. 대응방안 - Command Injection 공격에 대한 대응 방안
4. 레퍼런스


### 취약점 정보

![이미지](/assets/OWASP_injection.png)

2022년 부터 2023년 현재까지 OWASP(Open Web Application Security Project) 상위 10개 취약점에서는 Injection 공격이 3위를 이루고 있는 것을 확인할 수 있다. 취약점에 노출되어 주입 공격에 성공할 경우 악의적인 시스템 명령어를 통해 데이터 손실, 도난, 노출 등을 유발하며 최악의 경우 전체 시스템을 장악하고 손상시킬 수 있는 공격으로 심각한 피해를 입을 수 있는 공격이니 Injection에 대한 철저한 보안 설정 및 시큐어코딩이 필요하다.


### 취약점 설명
- Command Injection 이란?
명령어를 주입한다 라는 뜻으로, 취약한 애플리케이션을 통해 호스트 OS에서 시스템 명령을 실행하는 것을 목표로 하며 웹 요청 메시지에 신뢰할 수 없는 시스템 명령어를 삽입하여 검증 절차 없이 웹 서버에서 악의적인 명령어가 실행 되도록 하는 공격을 의미한다. 공격자가 사용하는 운영체제 명령은 일반적으로 취약한 응용 프로그램의 권한으로 실행된다. 

- 리눅스 메타문자
쉘에는 한 줄에 여러 명령어를 실행하는 등의 쉘 사용자 편의성을 위해 제공하는 특수 문자들이 존재한다. 시스템 명령이 수행된다면 특수문자를 활용해 Command Injection 공격이 가능하다.

| 메타문자 | 설명 |
| :------------------: | :-------------------------------------------------------------------------------------------------- |
| '' | 명령어 치환 : ''안에 들어있는 명령어를 실행한 결과로 치환 |
| && | 명령어 연속실행 : 한 줄에 여러 명령어를 사용하고 싶을 때 사용. 앞 명령어에서 에러가 발생하지 않아야 뒷 명령어 실행 |
| \| | 명령어 연속 실행 : 한 줄에 여러 명령어를 사용하고 싶을 때 사용. 앞 명령어에서 에러가 발생해야 뒷 명령어 실행 |
| ; | 명령어 구분자 한 줄에 여러 명령어를 사용하고 싶을 때 사용. ; 은 단순히 명령어 구분을 위해 사용하며, 앞 명령어의 에러 유무와 관계 없이 뒷 명령어 실행 |
| \|\| | 파이프 앞 명령어 결과가 뒷 명령어 입력으로 들어간다. |
| 그 외 | ., >, >>, &>, >&, <, {}, ?, *, ~ |


### 개념 증명
DVWA 취약 환경에서 Command Injection 공격을 실습해보았습니다. 다음과 같은 페이지에서 입력창에 IP를 넣으면 해당 IP주소로 ping 명령어를 실행한 후 그 결과를 출력해 주는 페이지이다.

#### * Low Level

![이미지](/assets/7_ping.png)

IP(127.0.0.1) 입력해 보았을 때 ping 127.0.0.1 결과를 출력해주는 것을 볼 수 있었다.

![이미지](/assets/1_id.png)

IP 뒤에 '&& id' 명령을 입력하였더니 ping 명령어 진행 후 id 명령을 통해 현재 공격 대상자의 사용자 정보를 출력하는 것을 확인하였다.

 만약 대상 시스템이 root 사용자 권한으로 실행되고 있는 경우 공격자는 root 권한을 쉽게 획득 할 수 있으며, root권한이 아니더라도 일반 사용자 권한에서 '권한 상승 공격' 을 시도가 가능하게 된다. 그렇게 되면 시스템을 장악할 수 있는 권한이 공격자에게 넘어가버려 큰 위험에 노출될 것이다.

![이미지](/assets/cat_etc.png)

IP를 생략하고 ';' 뒤에 'cat /etc/passwd' 명령을 통해 원격 호스트 대상으로 시스템 명령어를 자유롭게 사용 가능한 것을 확인할 수 있었다. 

리눅스 환경에서 '/etc/passwd' 파일은 시스템에 등록된 사용자 정보들이 담겨있는 파일로, 관리자는 해당 파일을 이용하여 사용자의 계정과 인증을 관리한다. 만약 시스템에 평문의 비밀번호를 가지고 있는 파일이 저장되어 있다면, 모든 사용자의 비밀번호가 노출될 수 있는 위험도 피할 수 없다.

#### * 소스코드 확인(Low)

사용자가 입력하는 인자값을 조작하여 OS 명령을 실행함으로써, 공격자가 원하는 정보를 획득할 수 있는 것을 확인하였다. Command Injection 취약점에 노출된 웹 환경의 소스코드를 확인해보았다.

```c
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

소스 코드를 확인 해보니 ping 명령을 전달할 때 'ping -c 4'를 통해 ping을 보내게 된다. -c 옵션은 4번의 ICMP request와 reponse로 ping 명령을 종료하겠다 라는 의미이다. 뒤에 '$target'은 웹의 요청 메시지로 부터 전달된 파라미터 값으로 '$target' 부분에 IP를 전달받아 ping 명령이 실행되게 된다.

따라서 IP값을 전달하여 ping 명령을 실행할 때 뒤에 메타문자(;,&&,\| 등)을 사용하여 다른 명령어가 실행되도록 전달인자 값을 보내어 시스템의 정보 노출 및 root 권한 획득 공격까지 수행할 수 있다.

#### * Midium Level

Midium Level에서는 동일하게 ';cat /etc/passwd' 명령을 실행해보았을 때 이전과 같이 사용자 정보 획득 공격에 성공할 수 없었다. 소스코드를 확인하여 우회할 수 있는 방안을 확인해보았다.

#### * 소스코드 확인(Midium)

<!--midium level의 소스코드-->
```c
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Set blacklist
    $substitutions = array(
        '&&' => '',
        ';'  => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

소스코드를 보니 ';', '&&' 메타문자를 사용할 경우 Command Injection이 실행되지 않도록 해당 문자가 지워지도록 시큐어코딩이 되어있었다. 

하지만 위 설명과 같이 ';', '&&' 문자 이외에도 우회할 수 있는 다른 메타문자가 존재한다. 

![이미지](/assets/midium_3.png)

위와 같이 '&' 사용하여 앞에 ping 명령을 백그라운드에서 사용되게 만들고 뒤에 id 명령을 통해 사용자의 권한을 획득하였다.

![이미지](/assets/midium_4.png)

이번에는 '\|' 파이프 문자를 통해 사용자 정보 획득까지 시큐어코딩을 우회하여 공격에 성공한 것을 확인할 수 있었다. 

#### * High Level

이 전에 사용하였던 문자들을 모두 실행해보았을 때 Command Injection 공격이 정상적으로 수행되지 않았다.

#### * 소스코드 확인(High)

<!--high level의 소스코드-->
```c
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = trim($_REQUEST[ 'ip' ]);

    // Set blacklist
    $substitutions = array(
        '&'  => '',
        ';'  => '',
        '| ' => '',
        '-'  => '',
        '$'  => '',
        '('  => '',
        ')'  => '',
        '`'  => '',
        '||' => '',
    );

    // Remove any of the charactars in the array (blacklist).
    $target = str_replace( array_keys( $substitutions ), $substitutions, $target );

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?> 
```

이 전과 비교하여 많은 문자를 제한하는 시큐어코딩으로 &, ;, \|, - 등 9개의 문자를 사용하는 경우 해당 문자를 공백으로 치환하여 대응을 하고있는 것을 볼 수 있다.

하지만 코드를 보면 개발자의 실수로 '\| ' 이 처럼 문자 뒤에 공백이 추가되어 있는 것을 확인할 수 있었다. 따라서 해당 문자 뒤에 '\|'를 하나 더 붙이게 되면 공백으로 치환되면서 뒤에 썼었던 '\| cat /etc/passwd' 명령어가 실행되었다.

'\|\|' 문자도 공백으로 치환하도록 되어 있는데 공격에 성공했던 이유는 '\|\|'을 실행하기 이전에 '\| ' 명령어 치환이 이미 수행되었기 때문에 대응을 마치고 다음 조건문으로 넘어갔기 때문이다.

이 처럼 잘 대응을 했지만, 하나의 실수로 공격자에게 정보가 노출 되는 것을 확인할 수 있었다.


### 취약점 원인

위와 같이 Command Injection에 취약점이 있는 웹 환경을 확인하였다. 어떻게 시스템 명령어를 호출하는 어플리케이션의 인자 값을 조작하고 Shell 권한을 흭득하여 의도하지 않은 시스템 명령어를 실행시키는 공격이 가능한걸까?

웹 어플리케이션 시스템에 system(), exec() 등과 같이 시스템 명령어를 실행시킬 수 있는 함수를 제공하게되면 공격자가 인자값을 조작하여 검증 절차 없이 운영체제 시스템 명령어를 호출하고 사용자 계정 정보 및 시스템의 관리자 권한 탈취하여 보안에 심각한 영향을 미치는 것이다.

###### Command Injection 공격에 취약한 함수

어플리케이션 별로 Command Injection 공격에 취약한 함수는 아래와 같다.

| 언어 | 설명 |
| -------- | -------------------------------------------------------------------------------------------------------- |
| Java | Ststem.*, System.runtime, Runtime.exec() |
| C/C++ | system(). exec(), ShellExecute() |
| python |exec(), eval(), os.system(), os.popen(), subprocess.popen(), subprocess.call() |
| Perl |open(). sysopen(), system(), glob() |
| php |exec(), system(), passthru(), popen(), rquire(), include(), eval(), preg_replace(), shell_exec(), proc_open(), eval() |


### 대응 방안

1. 웹 어플리케이션의 취약한 함수 사용을 최대한 피해야 한다.

2. 전달 인자값에 대한 형식 및 데이터의 길이 지정
- IP의 정해진 제한 범위인 0.0.0.0 ~ 255.255.255.255 처럼 미리 정의된 IP 형식과 데이터의 길이를 제한하여 인자값의 배열을 지정해 부적절한 외부 입력 값이 전달되지 못하도록 한다.

```c
    if( ( is_numeric( $octet[0] ) ) && ( is_numeric( $octet[1] ) ) && ( is_numeric( $octet[2] ) ) && ( is_numeric( $octet[3] ) ) && ( sizeof( $octet ) == 4 ) ) {
        // If all 4 octets are int's put the IP back together.
        $target = $octet[0] . '.' . $octet[1] . '.' . $octet[2] . '.' . $octet[3]; 
```

3. 최소 권한의 원칙을 사용하며 각 시스템 계정마다 적절한 권한을 부여하고, 사용자 정보가 노출될 수 있는 파일은 암호화 하여 공격이 수행되더라도 시스템에 피해를 최소화한다.


### 레퍼런스
- OWASP Top 10 에 대한 설명 : [https://www.edudwar.com/owasp-top-10-vulnerabilities/](https://www.edudwar.com/owasp-top-10-vulnerabilities/)
- Command Injection 공격에 대한 설명 및 대응방안 : [https://vmilsh.tistory.com/287](https://vmilsh.tistory.com/287)
