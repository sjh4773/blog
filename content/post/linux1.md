---
title: "리눅스 쉘스크립트(변수, 값, 문자)"
date: 2021-09-05T23:03:09+09:00
categories:
  - linux
tags:
  - linux
---

# 리눅스 쉘 스크립트

- 유닉스 환경에서 간단한 프로그램을 작성하거나, 유닉스 명령어들을 조합해서 사용할 수 있도록
  도와주는 프로그램 도구

#!/bin/bash

val="hello"

if [ $val == "hello" ]; then
   echo "hello"
elif [ $val == "hi" ]; then
   echo "hi"
else
   echo "else"
fi

## 변수
- 변수 선언은 "="를 사용

name="hero"
age=30

echo ${hero}
echo $age

- 변수 선언시 공백이 없도록 한다!!!
- name = hero(X)

## 변수

- $0 - The name of the Bash script.
- $1 - $9 - The first 9 arguments to the Bash script
- $# - How many argument were passed to the Bash script
- $@ - All the arguments supplied to the Bash script(전체 argument 리턴)
- $? - The exit status of the most recently run process(쉘 스크립트 내에서 어떠한 실행문을 수행하고 난 뒤 어떤 값을 리턴했는지를 말해줌)
- $$ - The process ID of the current script(리눅스 상에서 실행되는 모든 프로세스는 프로세스 ID를 가지고 있다. 현재 실행하는 프로세스 ID를 리턴해줌)

- $USER - The username of the user running the script
- $HOSTNAME - The hostname of the machine the script is running ond
- $RANDOM - Returns a differnt random number each time is it referred to

## 값이나 문자

- read [-ers] [-a array] [-d delim] [-i text] [-n nchars] [-N nchars]
  [-p prompt] [-t timeout] [-u fd] [name ...]

- read var1  // var1으로 값을 읽는다.

- read -n3 var1  // var1으로 값을 읽는다. (문자 3개만 입력 받음.)

- read -p "input data: " var1  // 문자열 출력하면서 var1 값을 읽는다.

- 예제 1
   - 3개의 문자열을 입력 받아서 출력한다.
   - 3개의 문자열(크기 : 알파벳 3글자)을 입력받아서 출력한다.

            #!/bin/bash

            echo "Enter String"
            read -n3 var1
            echo "Input is $var1"

            echo "Enter String"
            read -n3 var2
            echo "Input is $var2"

            echo "Enter String"
            read -n3 var3
            echo "Input is $var3"

- 예제 2
  - 입력 받은 문자열의 개수와 문자열 내용을 출력해보자.

            #!/bin/bash

            # my2.sg file
            # my2.sh arg1, arg2, arg3, ...

            echo = "Input String Count : $#"
            echo = "Input Strings : $@"
            echo = "Input String 1 : $1"
            echo = "Input String 2 : $2"
            echo = "Input String 3 : $3"
            echo = "Input String 4 : $4"


## 출처 : 유튜브, https://www.youtube.com/watch?v=RIKJVIcYNqE, YW KO