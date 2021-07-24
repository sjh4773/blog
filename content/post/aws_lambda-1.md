---
title: "AWS Lambda와 MongoDB 연동"
date: 2021-07-24T15:51:51+09:00
description: "AWS Lambda"
categories:
  - "AWS Lambda"
tags:
  - "AWS Lambda"
---


# AWS Lambda와 MongoDB 연동
- 게시판 API를 구현해서 정상적으로 동작시키기 위해서 MongoDB와 AWS Lambda를 서로 연동해서 사용해야한다. 
- 이를 위해서는 별도의 MongoDB가 사전에 구축이 되어있어야 한다.(Atlas MongoDB)




![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo2.PNG)


            

            


mongoDB Atlas에서 미리 만들어져있는 Clusters에서 connect 버튼을 누르면 나오는 화면에서







![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo2.PNG)






![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo3.PNG)
            
            
            



Connet Your Application 누르면 우리의 mongoDB에 접속할 수 있는 URL 주소가 나온다.





![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo4.PNG)

            

            

            이것을 copy 해서 AWS Lambda function에 환경 변수로서 설정해서 사용할 수 있다.
            환경변수를 설정한 뒤 환경변수를 Lambda 함수 안에 불러와 사용할 수 있다.
            
            게시판 게시글 API를 만들기 위해서 mongoDB와 연동하는 방법이다.
            
            일반적으로 mongoDB는 mongoose라는 라이브러리를 통해 손쉽게 mongoDB에 접속할 수 있다.
            
            const mongoose = require('mongoose');
            
            const MONGODB_URI = process.env.MONGODB_URI; 라고 넣어주면 환경변수로 설정했던 변수값을
            그대로 이용할 수 있다.
            
            왜 굳이 그냥 문자열을 코드상으로 붙여넣지 않고 환경변수를 이용하냐면 코드상으로 데이터베이스의 패스워드가
            노출이 되면 보안상 문제를 야기할 수 있으므로 이렇게 패스워드값은 환경변수르 설정하는 습관을 들여야 한다.



## 코드 설명

![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo5.PNG)

           
           
            

            
            handler 함수 내부에서는 데이터베이스 설정 및 변수 초기화라고 해서 초기화를 진행할 수 있도록 한다.
            먼저 이렇게 mongoose. Promise = global.Promise; 설정을 해주고
            그리고 mongoose.connect(MONGODB_URI, { useNewUrlParser: true}); 이 커넥트 함수를 이용해서
            mongoDB URI에 접속할 수 있도록 한다. 그리고 여기에서 useNewUrlParser라고 해서 여기에 true값을 넣어주면 된다.
            그리고 데이터 베이스 변수를 만들어주면 되는데 이 mongoose에 connection 객체를 불러와서
            데이터베이스에 접속할 수 있도록 하면 된다. 접속이 완료가 되었다면 log를 찍을 수 있도록 한다.
            'Database Connected'라고 출력하도록 만든다.
            
            코드를 테스트하면 에러가 발생하는데 moogoose라는 라이브러리가 기본적으로 aws lambda에 포함되지 않았으므로
            발생하는 에러이다. 따라서 라이브러리를 불러와야 한다.
            AWS Lambda는 기본적으로 몇가지 라이브러리를 제공하고 있지만 별도의 추가적인 라이브러리가 필요할 때는
            직접 라이브러리를 업로드 할 필요가 있다. 그때 가장 효과적으로 이용할 수 있는것이 Layer이다.
            따라서 node.js 라이브러리를 압축해서 AWS Lambda에 올릴 수 있도록 해야한다.
            
            별도의 하나의 폴더를 만들어 주고 폴더 경로를 복사해서 명령 프롬프트 사이에서 접속할 수 있도록 해야한다.
            노드 같은 경우 npm install mongoose 명령어를 이용해서 특정 폴더에 라이브러리를 설치할 수 있다,
            에러가 발생할 경우 npm init 명령어를 통해 초기화 한뒤에 다시 설치한다.
            
            폴더에 node_modules라는 폴더가 만들어지고 이 폴더안에 mongoose 관련 라이브러리가 설치되어 있는것을 확인할 수 있다,
            이제 이 설치된 라이브러리를 그대로 압축해서 올리면 된다. 이때 최상위 폴더 같은 경우에는 이미 폴더 이름이 정해져 있으므로
            nodejs라는 폴더를 만들어서 그 안에 라이브러리 폴더를 넣어주고 이제 하나의 폴더를 더 만들어서 이름을 Library라고 설정한 뒤에
            nodejs라는 폴더를 Library 폴더안에 넣는다. 그 다음에 Library 폴더를 압축하게 되면 이 압축파일의 최상위 폴더의 이름이
            nodejs로 설정되어 있는것을 확인할 수 있다 이것은 일종의 AWS Lambda에서 약속하고 있는 형식이기 때문에 이것을 따라줘야한다.
            nodejs안에는 node_modules가 들어가고 있고 그 안에 library들이 존재함을 확인할 수 있다.
            --> 오류 발생!!



![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo6.PNG)



            


            이제 Library 압축파일을 AWS Lambda에 업로드하면 된다. 이때 적용하는 런타임 환경이 Node.js 10.x 이므로
            적용하는 AWS Lambda 함수도 같은 런타임 환경을 가지고 있어야 한다.
            Layer의 런타임 환경과 AWS Lambda 런타임 환경을 일치시켜줘야 한다.
            
            
            const mongoose = require('mongoose');
            const MONGODB_URI = process.env.MONGODB_URI;
            
            exports.handler = async (event, context, callback) => {
                /* Database 설정 및 변수 초기화 */
                mongoose.Promise = global.Promise;
                mongoose.connect(MONGODB_URI, { useNewUrlParser: true});
                const db = mongoose.connection;
                db.once('open', function(){
                    console.log('Database Connected.');
                });
            };




![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo8.JPG)

            


            코드를 실행하였을 때 정상적으로 데이터베이스에 접근이 되지 않는다.
            데이터베이스에 접속을 시도하는 정보를 바르게 확인하기 위해서 위 코드에서 async를 지워주고 다시 save를 해서
            test를 한다. 그러면 이제 db에 접근했을 때 그 결과를 확실히 확인 할 수 있다.
            그러나 오류 메시지가 출력이 되고 이를 확인해보면 특정한 mongoDB 서버에 접속을 할 때 있어서 오류가 발생했다고 나온다.
            즉 접속할 수 없다고 메세지가 나온다. 이 오류가 발생하는 이유는 mongoDB는 기본적으로 white list 기반으로 데이터베이스를
            이용할 수 있도록 설정이 되어있다. 그렇기 때문에 현재 AWS Lambda라는 서비스에서 mongoDB에 접속을 하려고 시도를 했을 때
            이 AWS Lambda 서버의 IP주소가 mongoDB에 등록이 되어있지 않기 때문에 오류가 생기는 것이다. 이를 해결하기 위해 white list 관리 탭으로
            들어간다. mongoDB 같은 경우는 기본적으로 데이터베이스에 접속하고자 하는 모든 컴퓨터의 IP를 등록을 해야 정상적으로 해당 컴퓨터에서
            mongoDB에 접속을 할 수 있게 되는데 AWS Lambda 같은 경우에는 유동 IP정책을 따르고 있어서 일일이 모든 IP를 white list에 등록할 수 없다.
            흔히 VPC를 이용해 고정 IP를 할당할 수도 있는데 다만 그렇게 하면 어렵고 비용도 많이 발생할 수 있다. 따라서 그냥 실습 과정에서는
            mongoDB에서 모든 IP를 허용할 수 있도록 설정할 수 있다. 따라서 기존의 white list IP를 지워주고 새로운 IP Address를 추가해주는 란에서
            ALLOW ACCESS FROM ANIWHERE을 누르면 모든 IP 대역폭에서 해당 mongoDB에 접속할 수 있도록 할 수 있다.

## 결과

            const mongoose = require('mongoose');
            const MONGODB_URI = process.env.MONGODB_URI;
            
            exports.handler = (event, context, callback) => {
                /* Database 설정 및 변수 초기화 */
                mongoose.Promise = global.Promise;
                mongoose.connect(MONGODB_URI, { useNewUrlParser: true});
                const db = mongoose.connection;
                db.once('open', function(){
                    console.log('Database Connected.');
                });
                context.done(null, {'statusCode': 200}); # 처리된 결과를 완전히 종료된 형태로써 반환할 수 있도록 하기 위해서 context.done 함수를 불러와서 statusCode로 200번을 내보내도록 할 수 있음
            };




![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo9.PNG)






## 오류 발생 및 해결

            



            라이브러리 zip을 업로드 하고 코드를 테스트 하였을 때, AWS Lambda Error: “Cannot find module '/var/task/index'”
            다음과 같은 에러가 발생하였고 이를 https://stackoverflow.com/questions/41750026/aws-lambda-error-cannot-find-module-var-task-index를 참고하여 해결하였다.





![](C:\Hugo\blog\themes\mainroad\static\image\awsmongo7.PNG)



        



            내가 업로드 한 zip 파일은 AWS에서 요구하는 유효한 형식이 아니었다.
            AWS Lambda에서 요구하는 zip 파일의 형식은 zip을 두 번 클릭하였을 때 직접 코드 파일을 표시하기를 원한다.
            기존에 나는 Library -> nodejs -> node -> node_modules 의 형태로 zip 파일을 구성하여 AWL Lambda에서 인식하지 못한 것이다.
            따라서 나는 AWS Lambda에서 요구하는 형태로 zip 파일을 생성하기 위해
            1. cmd 열기
            2. node_modules가 있는 파일의 경로로 들어가기
            3. zip -r nodejs.zip * 명령어 수행
            이와 같은 과정을 통해 zip 파일을 생성하고 업로드를 하였더니 정상적으로 동작되었다.