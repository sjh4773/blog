---
title: "AWS Lambda로 게시판 서버 API 만들기"
date: 2021-07-25T15:31:14+09:00
description: "AWS Lambda"
categories:
  - "AWS Lambda"
tags:
  - "AWS Lambda"
---

            API Gateway에서 Board-API로 들어가 /board에 GET,POST,PUT,DELETE 메서드를 추가하여 해당 Lambda 함수에 접근할 수 있도록 API개발해야한다.
            Lambda Proxy integration을 적용하고 Project-Board 함수가 실행될 수 있도록 Lambda 함수와 연결.
            일반적으로 GET,POST,PUT,DELETE가 REST API에서 가장 많이 사용된다.
            일반적으로 REST API를 따르는 API를 개발할때는 그냥 /board라고 한다음에 이 /board 리소스에 GET 방식으로 접근하게 되면
            모든 게시글 정보를 다 화면에 출력하도록 만들고 일반적으로 특정한 게시글 하나에 대한 정보만 확인하고자 한다면
            PASS parameter라는 것을 이용하는데 PASS parameter는 경로로 parameter 값을 보낸다.
            이것을 구현하기 위해 /board에서 어떤 리소스에 접근할 것인지 설정할 수 있다.

![](/image/board1.png)

            아래 설명을 읽어보면 {proxy+}라고 하면 모든 리퀘스트를 받아서 처리할 수 있다는 것을 알 수 있다.
            위쪽의 Configure as proxy resource에 체크를 해준다.
            그리고 Resource Path에 {proxy+}라고 적어준다. 그리고 Resource Name에는 proxy라고 적어준다.
            이런 방식으로 리소스를 만들어준다.
            마찬가지로 /board에  포함되어 있는 어떠한 파라미터를 받을 때 그러한 접근 또한 마찬가지로
            Project-Board 함수에 매칭시켜준다.

            작성할 게시물 REST API에 기본적인 명세를 먼저 확인한다

            [ 게시물 ]

            - 게시물 번호
            - 게시물 작성자
            - 게시물 비밀번호
            - 게시물 내용
            - 게시물 작성일자


            하나의 게시물에는 게시물 번호가 있고 어떤 사람이 게시물을 작성했는지 게시물 작성자가 들어가고
            그리고 게시물 비밀번호, 게시물 내용, 마지막으로 게시물 작성일자까지 이렇게 기록이 된다고 가정한다.
            이러한 명세에 따라서 게시물 API를 만든다.

## 코드 설명

            const mongoose = require('mongoose');
            const MONGODB_URI = process.env.MONGODB_URI;

            /* Board 오브젝트를 정의합니다. */
            const boardSchema = mongoose.Schema({
                id: {
                    type: Number,
                    required: true
                    },
                    name:  {
                    type: String,
                    required: true
                    },
                    password:  {
                    type: String,
                    required: true
                    },
                    content:  {
                    type: String,
                    required: true
                    },
                    date:  {
                    type: Date,
                    required: true
                    }
            });

            /* 하나의 연결 객체를 반복적으로 사용합니다. */
            let connection = null;
            const connect = () => {
                if (connection && mongoose.connection.readyState === 1) {
                    return Promise.resolve(connection);
                }
                else {
                    return mongoose.connect(MONGODB_URI, { useNewUrlParser: true })
                    .then(
                        conn => {
                                connection = conn;
                                return connection;
                        }
                    };
                }
            };

            exports.handler = (event, context, callback) => {
                let operation = event.httpMethod;
                let Board = mongoose.model('board', boardSchema);
                let proxy, password;
                switch (operation) {
                    case 'POST':
                            /*
                                경로: /board
                                파라미터: {
                                    "name" : "작성자",
                                    "content" : "내용",
                                    "password" : "비밀번호"
                                }
                                설명: 특정 게시글을 작성합니다.
                            */
                            let lastId = 0;
                            // 가장 최근에 작성된 게시물 번호를 가져옵니다.
                    connect().then(() =>
                                Board.findOne({})
                                        .sort({id: -1})
                                        .exec(function(err, board) {
                                            if(err) {
                                                context.done(null, {
                                                    'statusCode': 500,
                                                    'body': err
                                        });
                                            } else {
                                                lastId = board ? board.id : 0;
                                                const { name, content, password } =
                                                    JSON.parse(event.body);
                                                const newBoard = new Board({name, content, password});
                                                newBoard.date = new Date();
                                                newBoard.id = lastId + 1;
                                                // 새로운 글을 등록합니다.
                                                newBoard.save(function(err, board) {
                            if(err) {
                                                        context.done(null, {
                                                                'statusCode': 500,
                                                                'body': err
                                                }); 
                                                        } else {
                                                        context.done(null, {
                                                                'statusCode': 200,
                                                                'body': JSON.stringify(lastId + 1)
                                                });                                                
                                                        }
                                                })
                                            }
                                        })
                            );
                    break;
                        default:
                            callback(new Error('Operation Error: "${operation}"'))
                }
            };



            가장 먼저 mongoose를 이용할 때 하나의 게시물과 같은 어떠한 모드를 정의해서 그 모델 그대로 데이터베이스에 접근할 수 있도록 처리를 한다.
            const boardSchema = mongoose.Schema를 사용해서 하나의 schema를 정의할 수 있고
            먼저 게시물 번호가 들어간다. id라고 해서 type을 숫자로 넣어주고 반드시 들어가야 된다고 required를 true를 설정한다.
            name 같은 경우는 게시물을 누가 작성했는지에 대한 정보이고 마찬가지로 반드시 들어갈 수 있도록 required를 true라고 설정한다.
            password 같은 경우는 마찬가지로 String 형태로 꼭 들어갈 수 있도록 설정한다.
            content 같은 경우도 마찬가지로 String 형태이고 꼭 들어갈 수 있도록 설정한다.
            마지막으로 date는 type이 Date이고 꼭 들어갈 수 있도록 설정한다.
            이렇게 하나의 boardschema라는 변수를 이용해서 board object를 정의했다.
            일반적으로 이렇게 AWS Lambda에서 실제로 mongoose 객체를 이용해서 데이터베이스에 접근할 때는 하나의 연결 객체를 반복적으로 사용하도록 설정한다는 특징이 있다.
            이렇게 connection 객체를 하나 만들어서 이 Lambda 함수가 공통적으로 사용할 수 있도록 하고
            이제 connection이라는 하나의 함수를 정의할 수 있다. mongoose가 현재 readyState 값으로 1을 가지고 있다면
            mongoDB에 연결되어 있는 connection 객체가 존재하는 상황이므로 Promise.resolve(connection)이라고 해서 connetion을 반환할 수 있도록 한다.
            그렇지 않은 경우 connection 객체가 mongoDB와 연결되어있지 않은 상태이므로 먼저 mongoDB와 연결되는 과정이 필요하다.
            그래서 MONGODB_URI를 이용해서 연결할 수 있도록 하고 연결된 정보를 이용해서 connection 값을 초기화 해주고
            그 다음에 connetion을 다시 리턴하는 방식으로 현재 연결 되어있지 않은 상태라면 연결을 해서 그 연결된 정보 즉 connection 객체를 반환할 수 있도록 하면 된 것이다.

            이제 실질적으로 사용자 요청에 맞게 게시글 정보를 저장하고 가져오는 등의 작업을 해보도록 한다.
            먼저 operation을 받아야 한다. 왜냐하면 REST API 형태를 개발하고 있기 때문에 Method가 어떻게 들어왔는지를 확인할 수 있어야 한다.
            그 다음 Board라는 이름의 객체를 초기화 한다.
            그리고 proxy와 password는 많이 전달받는 데이터이므로 위쪽에 정의를 한다.
            먼저 POST 부터 정의를 한다. 사용자가 데이터를 전송하고자 할 때 즉 하나의 게시물을 등록했을 때에 대한 API이다.
            어떻게 정의를 해줄거냐면 경로 같은 경우는 /board라고 할 수 있고 그 다음 사용자가 {"name" : "작성자", "content" : "내용", "password": "비밀번호"}
            이렇게 전달한다고 가정을 한다. name에다가 작성자 이름을 적고 어떤 게시물인지 그 내용을 적었다고 가정을 한다. 그 다음에 이 게시물의 비밀번호까지
            담아서 입력을 했다고 가정을 한다. 다시 말해서 특정 게시글을 작성하는 것이다. 즉 게시물 작성에 대한 API라고 할 수 있다.
            사용자로부터 name과 content, password를 받았으니까 가장 최근에 작성된 게시물 번호를 가져올 필요가 있기 때문에
            connect 함수를 불러온다. 그러면 이제 connection 객체가 반환이 된다. 이제 그것을 받아서 Board에서 findOne 함수를 이용할 수 있다.
            이 findOne 함수 같은 경우는 현재 mongoDB에 저장되어 있는 게시물 정보를 가져오겠다는 소리이고 하나의 게시물 정보를 가져올 때
            id를 기준으로 내림차순으로 가져오도록 만든다. 그러면 가장 마지막에 등록되어 있는 게시물 정보가 반환된다.
            그래서 실제로 findOne 함수를 실행할 수 있도록 한다. 이때 만약에 오류가 발생했다면 context.done() 함수를 이용해서 statusCode에 500을 담아서
            사용자에게 반환한다 그 다음에 내용으로는 에러정보를 그대로 반환할 수 있도록 하면 된다.
            이제 오류가 발생하지 않고 정상적으로 id를 가져왔다고 가정한다. 그런데 여기에서 현재 게시물의 정보가 하나도 존재 하지 않는 상황이라면
            즉 초기 상태의 데이터베이스라면 이 board가 null값이 될 것이다. 그렇기 때문에 null 값이 아닐경우 id 값을 담고 그렇지 않을 경우 0을 담아서
            첫번째 id 값을 정해주는 것이다.
            그 다음에 name, content, password를 각각 사용자로부터 전달받는다.
            따라서 event.body 사용자의 요청을 body에 담은 그 내용을 그대로 name, content, password를 파싱해서 받아온다.
            다시 말해서 사용자가 name,content,password 정보를 보냈다면 각각 파싱해서 담아주겠다는 의미이다.
            name, content, password를 이용해서 하나의 Board 객체를 만든 뒤에 date에는 현재 시간을 담고
            아이디에는 lastId + 1이라고 넣어준다. 그러면 1번 번호부터 시작을 하게 된다.
            또한 이렇게 name, content, password를 넣고 date와 id 또한 무조건 넣어주어야 하는데 왜냐하면 하나의 게시글에 정보에 대해서
            모델을 정의할 때 id, name, password, content, date가 전부 하나도 빠짐없이 반드시 필요하다고 정의를 했기 때문이다.
            이렇게 하나의 Board 객체를 만들어주었으니 새로운 글을 등록할 수 있도록 한다.
            이때는 save()함수를 이용해서 바로 mongoDB에 해당 Board 객체를 등록할 수 있다.
            만약에 오류가 발생했다면 마찬가지로 위에서 정의했던 오류 관련 출력함수를 그대로 넣어주면 되고
            그리고 오류가 발생하지 않았다면 200번 코드를 반환할 수 있도록 한다.
            일반적으로 이렇게 반환을 할때는 JSON 형태로 가장 마지막에 등록된 그 Id 정보를 그대로 반환을 하는 경우가 많다.
            이상으로 이렇게 host 방식으로 사용자가 접근을 했을때에 대한 처리를 해주었다.




            API gateway를 확인하면 POST 방식의 API 주소가 나와있다. 당연히 그냥 들어가면 오류가 발생한다
            왜냐하면 사용자로부터 name, content 등과 같은 게시물 정보를 받아야 되기 때문이다.
            따라서 Restlet과 같은 확장 프로그램을 이용해서 REST API를 테스트 할 수 있다.
            서버의 주소를 넣어주고 POST 방식으로 데이터를 넣어준다.

![](/image/board8.png)


            {
            "name" : "작성자",
            "content" : "내용",
            "password" : "비밀번호"
            }
            를 API에 POST 방식으로 보낸다.
            그랬더니 약간의 시간이 흐르고 "Internal Server Error"가 발생한다.
            이때 오류가 발생할 때 오류를 확인할 수 있는 방법이 다양한데 일단 가장 먼저 POST 방식 등 우리가 정의하지 않은
            다른 메소드로 접속했다면 그에 대해 처리하기 위해서 default라고 해서 우리가 허용하지 않은 operation으로 접속했을 때에 대한 처리를 해줄 수 있다. 
            default:
                callback(new Error('Operation Error: "${operation}"'))

![](/image/board3.png)

            다시 TEST를 해도 오류가 발생하는데 이럴때는 API gateway 서비스 사이에서 바로 테스트를 해볼 수 있다.
            POST 방식으로 한번 테스트를 해보기 위해 들어가서 Request Body에 작성하고자 하는 내용을 그대로 복사해서 Body로 보낸다.
            테스트를 했을 때 왜 오류가 발생했는지에 대한 정보가 빠짐없이 출력이 된다. 따라서 오류가 발생했을 때는
            API gateway에서 테스트 탭을 이용해서 테스트를 진행하는 것을 추천한다. 다만 지금 오류 메세지를 확인해보면 오류 코드가 502번인 것을
            확인할 수 있다. 이 502번 코드는 bad gateway라는 뜻으로서 일반적으로 어떠한 함수가 제한시간 안에 끝나지 않거나 했을 때 
            발생할 수 있는 오류인데 지금 보면 정상적으로 save 함수까지 불러와져서 데이터가 들어왔다는 것을 유추할 수 있는데
            그럼에도 불구하고 이 context.done 함수로 인해서 함수가 종료되지 않아서 이와 같은 오류가 발생하는 것을 알 수 있다.

![](/image/board4.png)

            실제로 Atlas mongoDB에 가서 collections 탭으로 이동을 해보면 실제로는 boards라고 하는 데이터셋에 데이터는 등록이 되어있는 것을 확인할 수 있다.
            다시 말해서 정상적으로 데이터는 들어갔지만 이 Lambda 함수가 종료되지 않아서 이러한 문제가 발생하는 것으로 알 수 있다.
            이제 이것은 최신 Node.js 같은 경우는 context.done 함수를 더이상 사용하지 않기 때문이다.
            이제 callback 함수를 사용하는 방식으로 context.done 함수를 바꿔주면 된다.



            const mongoose = require('mongoose');
            const MONGODB_URI = process.env.MONGODB_URI;

            /* Board 오브젝트를 정의합니다 */
            const boardSchema = mongoose.Schema({
                id: {
                    type: Number,
                    required: true
                },
                name: {
                    type: String,
                    required: true
                },
                content: {
                    type: String,
                    required: true
                },
                password: {
                    type: String,
                    required: true
                },
                date: {
                    type: Date,
                    required: true
                }
            });

            /* 하나의 연결 객체를 반복적으로 사용합니다 */
            let connection = null;
            const connect = () => {
                if (connection && mongoose.connection.readyState === 1){
                    return Promise.resolve(connection);
                }
                else {
                    return mongoose.connect(MONGODB_URI, { useNewUrlParser : true })
                    .then(
                        conn => {
                            connection = conn;
                            return connection;
                        }
                    );
                }
            };


            exports.handler = (event, context, callback) => {
            context.callbackWaitsForEmptyEventLoop = false;
            let operation = event.httpMethod;
            let Board = mongoose.model('board', boardSchema);
            let proxy, password;
            switch (operation) {
                case 'POST':
                    /*
                            경로: /board
                            파라미터: {
                            "name": "작성자",
                            "content": "내용",
                            "password": "비밀번호"
                            }
                            설명: 특정 게시글을 작성합니다.
                        */
                        let lastId = 0;
                        // 가장 최근에 작성된 게시물 번호를 가져옵니다.
                        connect().then(() =>
                            Board.findOne({})
                                .sort({id: -1})
                                .exec(function(err, board){
                                    if(err) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': err
                                        });
                                    } else {
                                        lastId = board ? board.id : 0;
                                        const { name, content, password } =
                                            JSON.parse(event.body);
                                        const newBoard = new Board({
                                            name, content, password
                                        });
                                        newBoard.date = new Date();
                                        newBoard.id = lastId + 1;
                                        // 새로운 글을 등록합니다.
                                        newBoard.save(function(err, board){
                                    if(err) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': err
                                        });
                                    } else {
                                        callback(null, {
                                            'statusCode': 200,
                                            'body': JSON.stringify(lastId + 1) 
                                        });
                                    }
                                });
                            }
                        })
                    );
                    break;
                case 'GET' :
                    /*
                        경로: /board
                        설명: 전체 게시글 정보를 불러옵니다.
                    */
                    if(event.pathParameters == null) {
                        let query = {};
                        if(event.queryStringParameters !== null){
                            if(event.queryStringParameters.name){
                                query.name = {
                                    $regex:event.queryStringParameters.name,
                                    $option: 'i'
                            };
                        }
                        if(event.queryStringParameters.content){
                                query.content = {
                                    $regex:event.queryStringParameters.content,
                                    $option: 'i'
                                }; 
                            }
                        }

                        // name과 content를 이용하여 검색한 결과를 내림차순으로 반환합니다.
                        connect().then(() => {
                        Board.find(query)
                            .select("-password")
                            .sort({id: -1})
                            .exec(function(err, boards) {
                                    if(err) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': err
                                        });
                                    } else {
                                        callback(null, {
                                            'statusCode': 200,
                                            'body': JSON.stringify(boards) 
                                        });
                                    }
                            });
                        });
                    }
                    /*
                        경로: /board/:id
                        설명: 특정 게시글 정보를 불러옵니다.
                    */
                    else {
                        proxy = event.pathParameters.proxy;
                        connect().then(() => {
                        Board.findOne({id:proxy})
                            .select("-password")
                            .exec(function(err, board) {
                                    if(err) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': err
                                        });
                                    } else if(!board) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': JSON.stringify("Board not found.")
                                        });
                                    } else {
                                        callback(null, {
                                            'statusCode': 200,
                                            'body': JSON.stringify(board) 
                                        });
                                    }
                            });
                        });
                    }
                    break;
                case 'PUT' :
                    /*
                        경로: /board/:id
                        헤더: password: "현재 비밀번호"
                        파라미터: {
                            "name": "작성자",
                            "content": "내용",
                            "password": "비밀번호"
                            }
                        설명 : 특정 게시글을 수정합니다.
                    */
                    proxy = event.pathParameters.proxy;
                    password = event.headers.password;
                    // 사용자가 입력한 번호의 게시물을 찾습니다.
                    connect().then(() => {
                    Board.findOne({id:proxy})
                        .exec(function(err, board) {
                                if(err) {
                                    callback(null, {
                                        'statusCode': 500,
                                        'body': err
                                    });
                                } else if(!board) {
                                    callback(null, {
                                        'statusCode': 500,
                                        'body': JSON.stringify("Board not found.")
                                    });
                                } else {
                                    if(board.password !== password) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': JSON.stringify("Password is not correct.")
                                        });
                                    } else {
                                        const { name, content, password } =
                                            JSON.parse(event.body);
                                        // 사용자가 입력한 데이터에 맞게 정보를 변경한다.    
                                        Board.findOneAndUpdate(
                                            {id:proxy},
                                            { name, content, password }
                                        ).exec(function(err, board){
                                            if(err) {
                                                callback(null, {
                                                    'statusCode': 500,
                                                    'body': err
                                                });
                                            } else {
                                                callback(null, {
                                                    'statusCode': 200,
                                                    'body': JSON.stringify('Success') 
                                                });
                                            }
                                        });
                                    }
                                }
                        });
                    });
                    break;
                case 'DELETE' :
                    /*
                        경로: /board/:id
                        헤더: password:"현재 비밀번호"
                        설명: 특정 게시글을 삭제합니다.
                    */
                    proxy = event.pathParameters.proxy;
                    password = event.headers.password;
                    connect().then(() => {
                    Board.findOne({id:proxy})
                        .exec(function(err, board) {
                                if(err) {
                                    callback(null, {
                                        'statusCode': 500,
                                        'body': err
                                    });
                                } else if(!board) {
                                    callback(null, {
                                        'statusCode': 500,
                                        'body': JSON.stringify("Board not found.")
                                    });
                                } else {
                                    if(board.password !== password) {
                                        callback(null, {
                                            'statusCode': 500,
                                            'body': JSON.stringify("Password is not correct.")
                                        });
                                    } else {
                                        // 사용자가 입력한 번호의 게시물을 삭제합니다.   
                                        Board.findOneAndRemove(
                                            {id:proxy}
                                        ).exec(function(err, board){
                                            if(err) {
                                                callback(null, {
                                                    'statusCode': 500,
                                                    'body': err
                                                });
                                            } else {
                                                callback(null, {
                                                    'statusCode': 200,
                                                    'body': JSON.stringify('Success') 
                                                });
                                            }
                                        });
                                    }
                                }
                        });
                    });
                    break;
                default:
                    callback(new Error('Operation Error: "${operation}"'));
            }
            };

            또한 이 경우에는 context.callbackWaitsForEmptyEventLoop = false; 값을 넣어주게 되면 Lambda 함수에서 callback이 호출되었을 때 함수가 종료가 된다.
            Restlet과 같은 API 테스트 도구를 이용해서 데이터가 등록이 되는 것을 확인할 수 있다.
            현재까지 데이터가 두개 들어가 있는 상태였으므로 새롭게 들어간 데이터의 아이디인 3번이 출력되는 것을 확인할 수 있다.
            다시 SEND 버튼을 눌러 API를 테스트 할 때마다 새롭게 데이터가 들어가는 것을 확인할 수 있다.
            다시 말해서 이와 같이 context.done 함수를 지워주고 context.callbackWaitsForEmptyEventLoop에 false를 설정함으로써 정상적으로 함수가 종료가
            될 수 있도록 설정을 한 것이다.
            다시 한번 mongoDB에 들어가 있는 데이터를 새로고침 해서 확인해보면 정상적으로 등록한 데이터가 잘들어가있는것을 확인할 수 있다.
            이렇게 새롭게 데이터를 추가할 때마다 새로고침을 해보면 데이터가 잘들어가있다.
            이와 같이 POST 방식의 함수 즉 데이터의 삽입 API는 구현이 완료된 것이다.
            마찬가지로 API gateway에서 테스트 하는 부분에서 Request Body를 기존의 데이터로 설정해서 테스트를 해보면 정상적으로
            API 동작해서 어떠한 오류가 나오지 않는 것을 확인할 수 있다.
            결과적으로 POST 방식의 메소드 즉 하나의 게시글 데이터를 작성하는 이러한 API에 대해서는 작성이 완료되었다.

![](/image/board5.png)

![](/image/board6.png)

            이어서 GET 방식의 메소드 또한 작업을 해주겠다.
            GET 방식의 경우에는 /board 경로에 접속을 했을 때 전체 게시글 정보를 불러온다는 특징이 있다.
            또한 여기에서는 event.pathParameters가 null 값일 때 즉 진짜 말그대로 board라는 경로에 접속했을 때에 대해서 처리를 해주고
            그렇지 않은 경우도 분리를 해서 처리를 해준다. 그렇지 않은 경우라면 특정한 id에 게시글 정보를 확인하고자 할 때라고 말할수 있다.
            이와 같이 두 가지로 나눠서 작성 할 수 있다.
            먼저 기본 설정인 전체 게시글 정보를 불러올 때는 query로 검색을 해서 특정한 이름이나 내용에 해당하는 데이터만 추출할 수 있도록 설정을 한다.
            query.name이라고 해서 mongoDB에서 조회할 때 조회 조건을 걸어준다. queryStringParameters 즉 query parameter에 담겨있는 name 값을
            기준으로 해서 조건을 걸어주고 대소문자를 구별하지 않고 특정한 사람이 작성한 글 즉 name을 기준으로 해서 검색할 수 있도록 설정을 한다.
            그리고 마찬가지로 content를 통해서 검색을 할 수 있도록 한다. content에 대한 query 또한 담을 수 있도록 설정을 한다.
            즉 이 query 안에 name과 content를 각각 설정할 수 있다는 것이다.
            이제 name과 content를 이용하여 검색한 결과를 내림차순으로 반환할 수 있도록 한다. 마찬가지로 connect 함수를 이용해서
            연결을 수행한 뒤에 이제 password 값을 제외하고 가져올 수 있도록 한다. 왜냐하면 각각의 게시글에는 그 게시글을 수정하거나
            삭제하기 위한 password 값이 포함되어있고 그냥 단순하게 find 함수를 통해 데이터를 가져오도록 하면 password 값까지
            포함되기 때문에 password 값은 제외하겠다고 설정을 하는 것이다. 그 다음에 id 값을 기준으로 내림차순으로 위에서부터 게시글 데이터를
            가져올 수 있도록 한다. 그리고 이렇게 가져오다가 에러가 발생한다면 에러에다가 null 값이 아닌 값이 들어가게 된다.
            오류가 발생했을 때 상태코드 500으로 오류의 내용을 출력하고 그렇지 않다면 가져온 Board 정보를 그대로 반환할 수 있도록 하면 된다.

            이를 GET 방식으로 TEST 하면 오류가 발생한다. 이럴때는 마찬가지로 다시 API gateway로 가서 GET 방식으로 다시 TEST를 수행하면 된다.
            오류 메세지로는 null 값을 가지는 데이터에서 name이라는 property를 읽어올 수 없다고 나온다. 오류를 확인해보니 index.js:106번 라인에 오류가
            있음을 확인할 수 있다. queryStringParameters가 null 값이기 때문에 거기에서 name을 가져올 수 없다는 소리이다.
            그렇기 때문에  한번 더 if문으로 묶어서 queryStrinParameters가 null값이 아닌 경우에 name과 content 값을 확인해서 query에 담아주겠다고
            설정할 수 있다.

            TEST를 다시 한번 수행하였을 때 에러가 발생하는데 이는 break를 생략해서 그런 것이다.
            break를 집어넣고 다시한번 TEST를 수행하면 정상적으로 데이터를 잘가져와서 출력하는 것을 확인할 수 있다.
            다시 말해서 break를 빼주는 실수를 하지않는다면 데이터를 정상적으로 가져오는 것을 확인할 수 있다.
            Restlet에서 데이터를 조회했을때에도 정상적으로 현재의 게시글에 포함되어있는 모든 게시글 정보를 출력해주는 것을 확인할 수 있다.

![](/image/board7.png)

![](/image/board8.png)

            이어서 특정 게시글 정보만 불러오는 것 또한 구현할 수 있다.
            이때는 pathParameter로 proxy라는 이름으로 데이터가 건너오기 때문에 이제 이를 기준으로 해서 데이터를 검색할 수 있도록 데이터를
            찾는 connection 함수 부분을 그대로 복사해서 붙여넣기를 해주고 데이터를 하나만 찾을거기 때문에 findOne함수를 사용하고 id 값으로는
            proxy로 설정해준다. id가 그대로 proxy에 담기게 되기 때문이다.
            검색한 결과가 board에 담기게 되고 오류가 발생하지 않았다면 board 정보가 출력되게 하면 되는데 else if를 넣어서 board 정보가 null값인 경우에
            해당 board id가 유효하지 않은 아이디이므로 오류 callback을 그대로 복사해서 붙여넣기 한다음에 오류 메세지를 출력할 수 있도록 하면 된다.

![](/image/board9.png)

![](/image/board10.png)

            PUT 방식 같은 경우는 일반적으로 데이터의 수정이 발생했을 때 그것을 처리할 수 있도록 해준다
            특정한 id에 대해서 PUT 방식으로 접근을 한다면 그 게시물을 수정하려고 시도한다고 볼 수 있다.
            또한 일반적으로 헤더에다가 비밀번호 값을 담아서 보낼 수 있다.
            이렇게 헤더에 password라고 해서 현재 게시글의 비밀번호를 보내야한다고 가정을 한다.
            그 다음 마찬가지로 파라미터에는 작성자 이름, 내용, 비밀번호가 이렇게 수정될 정보로서 담겨온다고 가정을 한다.
            즉 특정 게시물을 수정을 하는 API라고 할 수 있다.
            먼저 사용자로부터 어떤 id를 수정할 것인지 그 경로 parameter를 받아올 수 있고 password 값 또한 받아올 수 있다.
            header에 담겨있는 password를 가져올 수 있다.
            사용자가 입력한 번호의 게시물을 먼저 찾아야한다.
            해당 아이디로 검색을 하고 검색을 할 때 있어서 password까지 다 할 수 있도록 검색을 하고 이제 정상적으로 해당 board 즉 게시물 정보를
            찾을 수 있다면 사용자가 헤더에 담아서 보낼 비밀번호와 원래의 비밀번호가 일치하지 않는다면 마찬가지로 비밀번호가 일치 하지 않는다는
            메세지를 반환할 수 있도록 한다.
            이제 정상적인 비밀번호를 헤더에 담았다면 사용자가 보내준 name, content, password를 가져와야한다.
            사용자로부터 각각 name, content, password를 받아서 사용자가 입력한 데이터에 맞게 정보를 변경한다.
            그럴때는  Board.findOneAndUpdate를 사용해서 정보를 찾아서 변경할 수 있다.
            찾고자하는 그 정보는 해당 id 값과 name,content,password를 다음 parameter로 넣어서 해당 id 값의 게시물의 name, content, password를
            각각 변경할  수 있도록 한다. 이렇게 해서 함수를 실행하면 되고 이 실행한 결과를 function에서 처리할 수 있도록 한다.
            만약 오류가 발생했다면 오류에 대한 정보를 출력하면 되고 성공적으로 변경이 완료되었다면 Success 문자열이 출력되도록 할 수 있다.


![](/image/board12.png)


![](/image/board13.png)


![](/image/board14.png)


![](/image/board15.png)


![](/image/board16.png)


![](/image/board17.png)



            DELETE 같은 경우는 지우고자하는 경로에 해당하는 id 값을 넣어주고 헤더에는 password로 현재 password를 넣어주게 되면
            그 게시물을 삭제하게 되는 것이다.
            따라서 PUT과 마찬가지로 proxy와 password 정보를 다 받아와야한다.
            번호의 게시물을 찾은 다음에 password가 일치할 때에 한에서 사용자가 입력한 번호의 게시물을 삭제한다.
            이때는 Board.findOneAndRemove 함수를 사용한다. 특정 아이디 값을 삭제할 수 있도록 한다.