---
title: 힙합 가사 텍스트 마이닝 및 워드 클라우드
date: 2021-03-23 19:37:34
categories:
 - r
tags:
 - r
---
KoNLP 패키지 설치하기
현재 KoNLP 패키지가 CRAN에서 삭제된 상태라 install.packages()를 이용해 설치할 수 없습니다.
아래 코드를 순서대로 실행하면 깃헙 버전 KoNLP 패키지를 설치할 수 있습니다.

1. java와 rJava 설치
install.packages("multilinguer")
library(multilinguer)
install_jdk()

2. KoNLP 의존성 패키지 설치
install.packages(c("stringr", "hash", "tau", "Sejong", "RSQLite", "devtools"), type = "binary")

3. 깃헙 버전 KoNLP 설치
install.packages("remotes")
remotes::install_github("haven-jeon/KoNLP", upgrade = "never", INSTALL_opts=c("--no-multiarch"))
   
```r
# 힙합 가사 텍스트 마이닝

# 문자로 된 데이터에서 가치 있는 정보를 얻어 내는 분석 기법을
# '텍스트 마이닝'이라고 한다.
# 텍스트 마이닝을 할 때 가장 먼저 하는 작업은 문장을 구성하는 어절
# 들이 어떤 품사로 되어 있는지 파악하는 '형태소 분석'이다.


# 한글 자연어 분석 패키지인 KoNLP(Korean Natural Language Processing)
# 을 통해 한글 데어터로 형태소 분석을 할 수 있다.
library(KoNLP)
library(dplyr)


# 사전 설정하기
# KoNLP에서 지원하는 NIA 사전은 98만여 개의 단어로 구성
useNIADic()


# hiphop.txt는 멜론 차트 랩/힙합 부문 2017년 3월 둘째 주
# 50위 곡들의 가사로 구성되어 있습니다.

txt <- readLines("C:/Users/w/Desktop/Data/hiphop.txt")
head(txt)

library(stringr)

# * 참고 : str_replace_all()에 사용된 기호 \\W는 특수문자를 의미하는 '정규 표현식'입니다.
# 정규 표현식을 이용하면 문장의 내용 중 이메일 주소, 전화 번호처럼 특정한 규칙으로 되어 있는
# 부분을 추출할 수 있습니다.
txt <- str_replace_all(txt, "\\W", " ")

# 명사 추출하기
# * 참고 : extractNoun()은 출력 결과를 리스트 형태로 반환합니다.
# table(unlist(nouns))는 리스트로 되어 있는 nouns를 빈도 테이블로
# 변환하는 기능을 합니다.
extractNoun("대한민국의 영토는 한반도와 그 부속도서로 한다")

# 가사에서 명사 추출
nouns <- extractNoun(txt)

# 추출한 명사 list를 문자열 벡터로 변환, 단어별 빈도표 생성
wordcount <- table(unlist(nouns))

# 데이터 프레임으로 변환
df_word <- as.data.frame(wordcount, stringsAsFactors = F)

# 변수명 수정
df_word <- rename(df_word, word = Var1, freq = Freq)

# 자주 사용된 단어 빈도표 만들기
# nchar()를 이용해 두 글자 이상으로 된 단어만 추출
df_word <- filter(df_word, nchar(word) >= 2)

# 상위 20개 단어를 추출한다.
top_20 <- df_word %>%
  arrange(desc(freq)) %>%
  head(20)

top_20

# wordcloud 패키지를 이용하면 워드 클라우드를 만들 수 있다.
install.packages("wordcloud")

library(wordcloud)

# wordcloud 패키지를 설치하고 로드한 후 글자 색깔을 표현하는데 사용하는 패키지
library(RColorBrewer)

# Dark2 색상 목록에서 8개 색상 추출
pal <- brewer.pal(8,"Dark2")

# wordcloud()는 함수를 실행할 때마다 난수를 이용해
# 매번 다른 모양의 워드 클라우드를 만들어 낸다.
# 항상 동일한 워드 클라우드가 생성되도록
# wordcloud()를 실행하기 전에 set.seed()로 난수를 고정한다.
set.seed(1234)


# 워드 클라우드 만들기

wordcloud(words = df_word$word,  # 단어
          freq = df_word$freq,   # 빈도
          min.freq = 2,          # 최소 단어 빈도
          max.words = 200        # 표현 단어 수
          random.order = F,      # 고빈도 단어 중앙 배치
          rot.per = 1,           # 회전 단어 비율
          scale = c(4, 0.3),     # 단어 크기 범위
          colors = pal)          # 색상 목록

# 출력된 워드 클라우드는 많이 사용된 다어일수록 글자가 크고
# 가운데에 배치되며, 덜 사용된 단어일수록 글자가 작고
# 바깥에 배치되는 형태로 구성되어 있다.



# 단어 색상 바꾸기


# 단어 색상 목록을 이용하면 워드 클라우드의 단어 색을 바꿀 수 있다.

pal <- brewer.pal(9,"Blues")[5:9] # 색상 목록 생성
set.seed(1234)                    # 난수 고정

wordcloud(words = df_word$word,  # 단어
          freq = df_word$freq,   # 빈도
          min.freq = 2,          # 최소 단어 빈도
          max.words = 200        # 표현 단어 수
          random.order = F,      # 고빈도 단어 중앙 배치
          rot.per = 1,           # 회전 단어 비율
          scale = c(4, 0.3),     # 단어 크기 범위
          colors = pal)          # 색상 목록


```