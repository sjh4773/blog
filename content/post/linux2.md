---
title: "Linux 명령어 정리"
date: 2021-09-06T18:48:12+09:00
categories:
  - linux
tags:
  - linux
---

## 관리자 권한 얻기

- root 권한 실행(sudo)
  - $ sudo reboot
  - $ sudo halt

- root 사용자로 전환
  - $ sudo su - root

## 리눅스 파일 시스템

- root
  - home : 윈도우의 사용자 폴더와 같음, 사용자 추가할때마다 home에 디렉토리가 만들어짐
  - bin : 실행 파일을 두는 곳
  - mnt, media : 리눅스나 유닉스는 루트로부터 파생되는 특정 디렉토리에 장치들을 엮어쓰게됨. 
                     엮어쓰는 장치의 디렉토리로 과거에는 mnt(마운트)를 썼었음. 만약에 CD-ROM을 연결해 쓰고 싶을 때
                     CD-ROM 데이터를 어떻게 읽냐면 자기 혼자 루트를 갖는게 아니라 루트에서 어느 디렉토리에
                     연결해서 쓰게됨 그게 mnt 디렉토리이다. 요새는 media라는 디렉토리를 이용함. 
  - usr : 윈도우의 Program Files와 같음.
  - etc : 프로그램이 사용할 초기화에 대한 설정들을 두는 곳
  - var : 프로그램이 실행되면서 만들어지는 값들의 저장 공간
  - bin : 모든 유저가 사용하는 실행 파일을 두는 곳
  - sbin : adimin 계정이 사용하는 실행 파일을 두는 곳
  - tmp : 운영체제가 사용하든 프로그램이 사용하든 임시로 데이터를 풀었다가 삭제했다가 그런 부분이 필요하면 tmp를 사용
  - sys : 어떤 시스템에 대한 설정들이 있는 곳

## 파일 탐색을 위한 명령어들

- pwd : 현재 디렉토리 경로를 출력
- ls     : 디렉토리 목록 나열
- cd    : 디렉토리를 변경

- 상대경로 : cd var 또는 cd ./var
- 절대경로 : cd /usr/mygame/var

## 파일 시스템을 위한 명령어들
- mkdir : 디렉토리 생성
- rmdir : 디렉토리 삭제
- touch : 빈 파일 생성
- mv    : 파일 이동 / 변경
- rm    : 파일 삭제
- rm -r : 디렉토리 및 디렉토리 안쪽의 모든 파일 삭제
- rm -ri :  i를 붙여줌으로써 파일을 지울 때 내부 파일들의 삭제 여부를 물어보며 삭제함
- cp     : 파일 복사

## 파일 편집 및 관리
- 파일 편집기
   - VI 
      - i   : 입력
      - w : 저장
      - q  : 종료
      - o : 한줄 띄기
      - yy + p : 한줄 복사
      - y + p   : 한글자 복사
   - GNU nano
   - Emacs
   - ed/ex

## nano를 이용한 파일 편집
- nano <filename>

## 파일 찾기와 파일 정보 확인
- find ./ -name <filename> size +1c --> 1바이트보다 큰 파일을 찾아라는 의미
- cat <filename> --> 파일 정보 확인
- head -n2 Hello.java --> 위에서부터 두 줄 확인
- tail -n2 Hello.java --> 아래서부터 두 줄 확인 
- grep <찾고자 하는 글자> <filename> --> 찾고자 하는 글자 확인
- grep <찾고자 하는 글자> <filename> --> 소대문자 구별 x
- ls Hello[12].java --> Hello1.java 또는 Hello1.java 파일이 있는지 확인
- diff <filename1> <filname2> --> 차이가 있는 줄 명시
- cmp <filename1> <filename2> --> 차이점 출력

## 유용한 명령어
- clear :  화면의 글자들을 제거
- history : 지금까지 다뤘던 명령어가 목록화 되어서 출력됨. !<index> 를 입력하면 명령어 실행가능.
- echo : 글자 등을 화면에 출력하기 위한 명령어
- echo "<문자>" >> <file> : 문자가 파일에 입력된다.
- redirection(>) : ex) echo "Okay" > test --> test파일에 "Okay"문자 저장, >> 사용하면 기존 파일에 내용 추가


## 압축 파일 다루기

### 압축하기
- tar -cf name.tar a b c    tar -zcf name.tar.gz a b c

### 압축풀기
- tar -xvf name.tar          tar -zxvf name.tar.gz

- f : 파일 이름을 지정
- c : 파일을 tar로 묶음
- x : tar 압축을 풂
- v : 내용을 자세히 출력
- z : gzip으로 압축하거나 해제함
- t : 목록 출력
- p : 파일 권한을 저장
- C : 경로를 지정

## 링크 파일 만들기
- ln -s test1.txt test1.ln // Symbolic Link, 바로가기
- ln test1.txt testln.txt   // Hard Link, 별칭

## 사용자 관리하기

### 고전적인 사용자 관리 명령어 : 홈디렉토리 수동생성
- useradd - 사용자 추가
- usermod - 사용자 변경
- userdel - 사용자 삭제

### dragon 사용자 추가하기
- $useradd dragon
- $ cat /etc/passwd 또는 tail -n2 /etc/passwd

            newlec:x:1000:1000:newlec park,,,:/home/newlec:/bin/bash
            dragon:x:1001:1001::/home/dragon

            $sudo passwd dragon

## dragon 사용자로 로그인하기
### 가상 터미널
- Alt + F2~6

##  Home directory 소유자 문제
### 디렉토리 소유자 정보

            ~$ls -l ../
            drwxr-xr-x 3 root root 4096 Jul 14 2017 dragon // root -> 소유자
            drwxr-xr-x 11 newlec newlec 4096 Jan 12 00:56 newlec

### 소유자 변경하기
            ~$sudo chown dragon ../dragon
            ~ls -l ../
            drwxr-xr-x 3 dragon root 4096 Jul 14 2017 dragon // dragon -> 소유자
            drwxr-xr-x 11 newlec newlec 4096 Jan 12 00:56 newlec

            ~$sudo chown dragon(소유자):dragon(소유그룹) /home/dragon --> 소유자와 소유그룹을 한번에 바꿀 수 있음.

## 사용자를 추가하는 스크립트 파일 생성
### adduser
#### ~/bin/adduser

            useradd testuser
            tail -n2 /etc/passwd
            mkdir /home/testuser
            chown testuser:testuser /home/testuser
            echo "testuser user added"

## 파일 권한 변경

            소유자(rwx),소유그룹(rwx),Other(rwx)
            -rwxr-xr-x text.txt -> 세개씩 묶여서 소유자에 대한 권한, 소유그룹에 대한 권한, other에 대한 권한을 나타냄. 맨 앞은 파일의 타입

## 사용자 관리
### 사용자 추가 옵션들

- -c : 사용자 설명 설정
- -s : 기본 쉘 설정
- -r : UID 설정(500 이하로)
- -d : 사용자 디렉토리 설정
- -g : 기본 그룹
- -G : 추가 그룹

            # useradd -d /home/username -s /bin/bash username
            # useradd -r number -c "..." -g groupname -G groupname -d /home/username -s /bin/bash username
            # useradd dragon
            # passwd dragon

### 향상된 사용자 추가 명령어

            # adduser dragon
            # passwd dragon

### 사용자 제거하기
            # userdel -r username

### 사용자 추가
- useradd(or adduser)

### 사용자 제거
- userdel(or deluser)

### 사용자 정보 수정
- usermod
 
## 사용자 프롬프트 설정하기
### 프롬프트 스트링(PS1) 출력하기

            newlec@newlec-server:~$ echo $PS1
            \[\e]0;\u@\h" \w\a\]${debian_chroot:+($debian_chroot)}\u@\h:\w\$

### 프롬프트 스트링(PS1) 변경하기

            newlec@newlec -server:~$ PS1="newlec : ";
            newlec : ls
            aa download ohho workspace

### 프폼프트 스트링의 이스케이프 문자

            \H : the hostname, \h : the hostname up to the first '.', \d : the date
            \n : newline, \r : carrage return,
            \t : the current time in 24-hour, \T : the current time in 12-hour

## 출력 색상 변경하기
### 색상 변경 예
            ~$ LS_COLORS="di=0;33"
### 파일 종류명
            di = directory
            fi = file
            ln - symbolic link
            pi = fifo file
            so = socket file
            bd = block (buffered) special file
            cd = character (unbuffered) special file
            or = symbolic link pointing to a non-existenet file (orphan)
            mi = non-existent file pointed to by a symbolic link (visible when you type ls -l)
            ex = file which is executable (ie. has 'x' set in permissions)

### 텍스트 형식
            0 = default colour
            1 = bold
            4 - underlined
            5 = flashing text
            7 = reverse field

## 리눅스 부팅 순서
### 초기화 스크립트 작성 위치
            1) BIOS
            2) Masster Boot Record (MBR)
            3) LILO or GRUB
            4) Kernel
            5) init : process number 1(PID=1)
            -/linuxrc: load modules / initialize devices / exits
            -/sbin/init
                        -/etc/inittab : run boot scripts
                                    -/etc/init.d/rcS
                                                -/etc/rcS.d/S* scrpts
                                                -/etc/rc.boot/*
            6) Run Levels

## 컴퓨터 시작과 종료 명령어
### 종료 방법

            $poweroff
            $shutdown
            $halt
            $init 0

### 재시작

            $reboot -> 다른 사용자가 사용중일 때는 root 계정으로 강제 재부팅
            $shutdown
            $init 6

### 프로필과 환경변수 설정방법
#### 우분투에서 환경 변수를 설정하기 위한 대표적인 파일들

            /et/environment
            /etc/profile
            /etc/profile.d/*.sh
            ~/.profile