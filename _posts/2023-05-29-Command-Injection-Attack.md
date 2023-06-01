## Command Injection 공격 (명령어 주입 공격)

### 개요
이번 Writeup 에서는 Command Injection(명령어 주입 공격) 공격에 대한 공부 및 실습을 진행해보았습니다. 실습은 취약한 웹 환경인 DVWA(Damn Vulnaerable Web Application)을 이용했으며, 모든 실습 및 공격은 본인이 소유한 가상머신에서 이루어졌습니다.

이번 Writeup에서 다룰 Command Injection과 관련된 공부는 다음과 같습니다.
1. 취약점 정보/설명 - Command Inejction 공격에 대한 연구
2. 개념증명 실습 - Burpsuite 툴을 사용한 공격 실습
3. 대응방안 공부 - Command Injection 공격에 대한 대응 방안

### 취약점 정보


### 취약점 설명
Command Injection 이란?
명령어를 주입한다 라는 뜻으로, 취약한 애플리케이션을 통해 호스트 OS에서 시스템 명령을 실행하는 것을 목표로 하며 웹 요청 메시지에 임의의 시스템 명령어를 삽입하고 전송하면 웹 서버에서 해당 명령어를 살행하도록 하는 공격을 의미합니다. 

![이미지](/assets/OWASP_injection.png)

2023년 OWASP(Open Web Application Security Project) 상위 10개 취약점에서는 Injection 공격이 3위를 이루고 있는 것을 확인할 수 있습니다. 취약점에 노출되어 주입 공격에 성공할 경우 악의적인 시스템 명령어를 통해 데이터 손실, 도난, 노출 등을 유발하며 최악의 경우 전체 시스템을 장악하고 손상시킬 수 있는 공격으로 심각한 피해를 입을 수 있습니다.

- Command Injection에 취약한 함수
OS에서 시스템 명령을 실행하여 데이터를 노출, 손실 및 도난이 가능하다 하였는데 해당 시스템 명령을 통해  Command Injection 공격에 취약한 함수와 메타문자 등이 존재합니다.

언어
Java - Ststen.*, System.runtime, Runtime.exec()
C/C++ - system(). exec(), ShellExecute()
python - exec(), eval(), os.system(), os.popen(), subprocess.popen(), subprocess.call()
Perl - open(). sysopen(), system(), glob()
php - exec(), system(), passthru(), popen(), rquire(), include(), eval(), preg_replace(), shell_exec(), proc_open(), eval()

- 메타문자
쉘에는 한 줄에 여러 명령어를 실행하는 등의 쉘 사용자 편의성을 위해 제공하는 특수 문자들이 존재합니다. 시스템 명령이 수행된다면 특수문자를 활용해 Command Injection 공격이 가능합니다.

메타문자 - 뜻
'' - 명령어 치환 : ''안에 들어있는 멸영어를 실행한 결과로 치환
$() - 명령어 치환 : $() 안에 들어있는 명령어를 실행한 결과로 치환, 중복 사용 가능
&& - 명령어 연속실행 : 한 줄에 여러 명령어를 사용하고 싶을 때 사용. 앞 명령어에서 에러가 발생하지 않아야 뒷 명령어 실행
|| - 명령어 연속 실행 : 한 줄에 여러 명령어를 사용하고 싶을 때 사용. 앞 명령어에서 에러가 발생해야 뒷 명령어 실행
; - 명령어 구분자 한 줄에 여러 명령어를 사용하고 싶을 때 사용. ; 은 단순히 명령어 구분을 위해 사용하며, 앞 명령어의 에러 유무와 관계 없이 뒷 명령어 실행
| - 파이프 앞 명령어 결과가 뒷 명령어 입력으로 들어간다.
그 외 - ., >, >>, &>, >&, <, {}, ?, *, ~


### 개념 증명
DVWA 취약 환경에서 Command Injection 공격을 실습해보았습니다. 다음과 같은 페이지에서 입력창에 IP를 넣으면 해당 IP주소로 ping 명령어를 실행한 후 그 결과를 출력해 주는 페이지 입니다.

![이미지](/assets/7_ping.png)

Burpsuite Intruder

### 취약점 원인

### 대응 방안

# 툴 제작 ?

# 레퍼런스
