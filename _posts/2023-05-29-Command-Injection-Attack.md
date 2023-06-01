## Command Injection Attack (명령어 주입 공격)

# 개요
이번 Writeup 에서는 Command Injection(명령어 주입 공격) 공격에 대한 공부 및 실습을 진행해보았습니다. 실습은 취약한 웹 환경인 DVWA(Damn Vulnaerable Web Application)을 이용했으며, 모든 실습 및 공격은 본인이 소유한 가상머신에서 이루어졌습니다.

이번 Writeup에서 다룰 Command Injection과 관련된 공부는 다음과 같습니다.
1. 취약점 정보/설명 - Command Inejction 공격에 대한 연구
2. 개념증명 실습 - Burpsuite 툴을 사용한 공격 실습
3. 대응방안 공부 - Command Injection 공격에 대한 대응 방안

# 취약점 정보


# 취약점 설명
Command Injection 이란?
명령어를 주입한다 라는 뜻으로, 취약한 애플리케이션을 통해 호스트 OS에서 시스템 명령을 실행하는 것을 목표로 하며 웹 요청 메시지에 임의의 시스템 명령어를 삽입하고 전송하면 웹 서버에서 해당 명령어를 살행하도록 하는 공격을 의미합니다.

# Linux 시스템 명령어 간단 설명


# 개념 증명
DVWA 취약 환경에서 Command Injection 공격을 실습해보았습니다. 다음과 같은 페이지에서 입력창에 IP를 넣으면 해당 IP주소로 ping 명령어를 실행한 후 그 결과를 출력해 주는 페이지 입니다.
![이미지](/assets/7_ping.png)

Burpsuite Intruder

# 취약점 원인

# 대응 방안

# 툴 제작 ?

# 레퍼런스
