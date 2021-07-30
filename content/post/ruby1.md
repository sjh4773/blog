---
title: "CRUD 게시판 만들기"
date: 2021-07-30T14:13:56+09:00
categories:
  - "Ruby on Rails"
tags:
  - "Ruby on Rails"
---

## MVC 패턴 설명
### 루비 온 레일즈를 이용해 게시판을 만들면서 MVC 패턴을 만들어보고 DB와 연결해 CRUD를 실습

![](/image/ruby1.png)

            우리가 브라우저에서 jocoding.com/board라고 붙이면 Router에서는 이 board를 읽어서
            Controller에게 게시판이 왔다고 전달해준다.
            그러면 Controller에서는 Model을 통해 DB 정보를 불어와서 게시글 정보를 가져온다,
            그리고 View를 통해서 그 게시물에 HTML, CSS 요소를 첨가해 준 뒤에 우리가 보는 화면으로 전송해준다.


## POST

![](/image/ruby2.png)



            우리가 만든 Form 화면이 Client Side에 떠 있을 텐데
            우리가 제목과 내용 정보를 채워주고 /create라는 주소로 보내줬습니다.



![](/image/ruby3.png)


            라우터에서는 /create로 날아온 것을 post 방식으로 board(Controller)에
            create(View=Action)라는 action으로 연결해준다.


![](/image/ruby4.png)

            그러면 BoardController에서는 create라는 액션으로 들어왔으니
            코드 내에서 def create로 가리키게 되고 함수 내의 코드가 수행된다.
            이때 파라미터로 날아온 title과 content를 받아서 변수에 넣고
            그리고 Post라는 Model을 불러와서 create라는 명령어를 통해 DB에 정보를 입력하게 된다.
            그리고 다시 /board라는 주소로 가게 함으로써 다시 index 화면으로 돌아가게 된다.



## READ

![](/image/ruby5.png)

            index 화면에 모든 게시물의 정보를 다 뿌려 준다.
            그러려면 @posts라는 변수를 만들고 여기안에 Post라는 Model를 불러와서 Post.all을 해주면
            그러면 Post라는 Model의 모든 것이 선택되서 @post라는 변수에 담긴다.


![](/image/ruby6.png)

            <%=화면에 표시 %>, <% 내부적으로 처리 %>

            @posts 안에는 게시물 여러 개가 있는데 여기에 .each를 통해서 하나씩 뽑아서 user라는 변수에 넣겠다는 뜻이다.
            그리고 do end 사이에 여기에 post를 입력하면 post(게시물 하나)를 쓸 수가 있다.

            점을 찍는 것은 앞 요소 안에 어떤 것을 사용할 때 씀

            post라는 데이터가 날라와서 post 안에 있는 title을 찍어주고 post안에 있는 content를 찍어준다.


## UPDATE

![](/image/ruby7.png)

            View에서 각 게시물에 <a> 태그로 "수정하기"라는 버튼(링크)를 만들어준다.
            그리고 href로 /edit 이라고 정해서 수정하는 페이지로 이동하게 해준다.
            대신 이게 몇 번 글인지 알아야 되기 때문에 post.id(Primary Key, 유일한 row 구분 가능)를 담아서
            Primary Key는 유일한 번호이기 때문에 번호를 통해서 어떤 글인지 식별할 수 있다.
            이 번호를 주소와 함께 보내준다. 그런데 여기는 <a> 태그이기 때문에(form이 아니라)
            <input>(처럼 정보를 name을 보내는)을 입력할 수 없다.
            따라서 "/edit?id=<%= post.id %>"를 입력하게 되면 id라는 파라미터로 값이 전달되게 된다.(input name="id" value="<%=post.id%>와 같음)

![](/image/ruby8.png)

            get 방식으로 edit이라고 날아오면 board에 edit으로 이동하도록 연결해준다.

![](/image/ruby10.png)

            그리고 Controller에 똑같이 edit을 만들고 edit은 수정화면이니까 수정할 게시물 하나만 보이면 된다.(1개의 record)
            그러기 때문에 @posts가 아니라 @post로 고쳐주고 Post Model에서 .all이 아니라 하나만 가지고 오면 되는데
            Primary Key(id) 값은 유일한 값이므로 Post(전체 중)에서 하나의 글만 찾아올 수 있다.
            find() 함수를 이용한다.


![](/image/ruby9.png)
            
            edit은 수정화면이니까 View도 만들어 준다.


            개발자 도구를 눌러서 수정하기 버튼을 보면
            각각으 edit 뒤에 id가 붙어서 전달되는 것을 볼 수 있고
            수정하는 페이지로 들어와서 태그를 보면 <Form> 태그를 보면 id라는 숨겨진 태그가 있는데
            그 안에 value에 1이 잘전달된 것을 확인할 수 있다.

            수정을 하기 위해 routes.rb에 get '/update', to: 'board#update' 라우팅 추가
            Controller에서 update를 추가하고 id를 이용해서 포스트를 찾는다.
            찾은 포스트를 이용하여 @post.title을 우리가 받은 파라미터의 title로 교체해주고
            마찬가지로 @post.content를 파라미터로 받은 content로 교체해준다.
            그리고 @post.save를 통해 post를 저장해준다.
            마찬가지로 redirect_to '/board' 로 설정해줘서 원래 화면으로 돌아가게끔 만들어준다.

## DELETE

            View에서 수정하기 밑에 똑같은 "삭제하기" 버튼을 만든다.
            마찬가지로 id를 넘겨주고 라우터에서 edit처럼 a 태그로 가기 때문에 get 방식으로 delete로 넘겨준 뒤 board에 delete라는 View로 연결해준다.
            그리고 Controller에서 마지막으로 delete라는 액션을 만들면 마찬가지로 post에서 id로 하나의 post를 찾은 다음에 그 post를 post.destroy라고하면
            post가 삭제 된다.


## 최종결과

![](/image/ruby11.png)

### 출처 : https://www.youtube.com/watch?v=2CHkds265N8,조코딩