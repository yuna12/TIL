# TIL #1  Full Text Search와 LIKE의 차이

길벗 출판사의 `똑똑하게 코딩하는 법 SQL 코딩의 기술`을 읽고 있다. 
1장 데이터 모델 설계를 읽고 있던 중에 
`두 개 이상의 컬럼을 기본키로 설정하는 복합 기본키(Compound Primary Eky)는 효율성이 좋지 않으므로 사용하지 않는 것이 좋다.` 면서 다음의 2가지 이유를 들었다.

```
1. 기본키를 정의할 때 대부분의 데이터베이스 시스템은 해당 컬럼에 유일 인덱스를 같이 만든다. 
   컬럼 두 개 이상에 유일 인덱스를 만들면 데이터베이스 시스템이 할 일만 더 많아 진다.
2. 일반적으로 기본키로 조인을 수행하는데, 기본키가 여러 컬럼으로 구성되어 있으면 쿼리가 좀 더 복잡하고 느려진다.
```

여기서 궁금증이 생겼다. 다중 컬럼 인덱스를 만들면 디비에서는 어떤 일을 더 해야 하는가.
구글링을 하다가 어떤 블로그의 글을 발견했다. (https://code-factory.tistory.com/24)

이 글은 인덱스를 효과적으로 사용하기 위해 생각해야 할 부분과 인덱스 동작들을 설명한다.
그 중에서 나는 아래와 같은 부분을 발견했다.
```
Index를 생성할 때 가장 효율적인 자료 형은 정수형 자료이다.
가변적인 크기와 정규화 할 수 없는 데이터는 인덱스를 생성할 때 비효율적으로 동작한다(text 데이터 등).
이를 위해 FULLTEXT Index를 사용할 수 있다.
FullTEXT는 TEXT, VARCHAR 등 가변적이고 일반적인 Index의 효율성이 떨어지는 부분에서 많은 효과를 가져올 수 있는 인덱스이다.
FULLTEXT Index는 텍스트 필드에 '%검색문자열%'과 비슷한 형태의 검색 결과를 얻을 수 있고 TEXT에 최적화된 Index 방식이다.
```

바로 여기에서 오늘 내가 공부 할 주제를 **Full Text search**으로 정하였다.

Full text index 동작은 이 블로그 글(https://interconnection.tistory.com/95) 에 설명이 잘 되어있다.

Full text index가 데이터를 인덱싱 하는 기법에는 1) Stop-word parser(built-in parser) 와 2) N-gram parser 이 있으며,
파싱 과정과 검색 과정이 예시와 함께 설명되어 있으니 인덱스 동작 방법과 검색 과정이 궁금하면 이 블로그를 참고해도 될 것 같다.

먼저, Full text search를 사용하는 방법은 다음과 같다.
(이 [블로그](https://kmongcom.wordpress.com/2014/03/28/mysql-%ED%92%80-%ED%85%8D%EC%8A%A4%ED%8A%B8fulltext-%EA%B2%80%EC%83%89%ED%95%98%EA%B8%B0/)의 글을 참조하여 정리하였다.)

**FULLTEXT 인덱스 생성**

: 테이블 생성시 컬럼에 FULLTEXT KEY 지정 또는 alter table을 이용한 FULLTEXT KEY 등록

(*주의!* *한글검색은 utf-8만 가능, text, binary char, varchar 컬럼 타입만 FULLTEXT KEY를 지정 가능,
  MyISAM 엔진을 사용하는 테이블에 대해서만 생성할 수 있음*)

1. 테이블 생성시 FULLTEXT KEY 등록

```sql
create table posts(
    id bigint(100) NOT NULL AUTO_INCREMENT,
    title text NOT NULL,
    date datetime NOT NULL,
    ....
    PRIMARY KEY (id),
    FULLTEXT KEY title (title)
) ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

2. alter table로 이미 생성된 테이블에 FULLTEXT KEY 등록

```sql
alter table posts add FULLTEXT(title);
```
여러 컬럼에 대해서도 FULLTEXT KEY를 생성할 수 있다.

```sql
alter table posts add FULLTEXT(title, body, keyword);
```

**자연어 검색**

: Full text search는 다음 쿼리를 사용합니다.
```{sql}
"SELECT * FROM "테이블명" WHERE MATCH("검색할컬럼명"[, ...]) AGAINST('"검색할키워드식" "검색모드");
```
검색 모드는 세 가지가 있으며, 1) 자연어 검색, 2) 불린 모드 검색(boolean mode search), 3) 쿼리 확장 검색으로 구분된다.

* 자연어 검색
```sql
SELECT * FROM posts WHERE MATCH (body) AGAINST ('Foo' IN NATURAL LANGUAGE MODE);
```
* 불린 모드 검색
```sql
SELECT * FROM posts WHERE MATCH (body) AGAINST ('+Foo*+bar*' IN BOOLEAN MODE);
```
* 쿼리 확장 검색
```sql
SELECT * FROM posts WHERE MATCH (body) AGAINST ('Foo*' WITH QUERY EXPANSION);
```

이제 본론으로 넘어가서 `Full Text Search와 LIKE의 차이`를 정리해보도록 하겠다. 

이쯤되니 궁금해지지 않겠는가.. % 연산자를 이용해서 LIKE로 검색하는 것과 Full Text Search로 검색하는 것의 차이는 무엇일까

마찬가지로 구글링해보니 스택오버플로우에 [나와 같은 질문](https://stackoverflow.com/questions/224714/what-is-full-text-search-vs-like)을 한 사람이 있었다.

잘 정리된 답변이 있어 여기에 정리 한다.
(출처: [스택오버플로우](https://stackoverflow.com/questions/17796717/fulltext-search-vs-standard-database-search/17796826#17796826))

full text searching의 이점은 다음과 같다.

**Indexing:**

아래와 같이 %연산자를 키워드 앞에 쓰고 LIKE 연산자를 사용하는 경우가 있다.

`WHERE Foo LIKE '%Bar';`

이러한 조건인 경우에는 `Foo` 컬럼이 인덱싱 된 컬럼일지라도 인덱스를 이용할 수 없다. 모든 행을 살펴보고 키워드와 일치하는지 확인해야 한다.
그러나 Full text index는 가능하다. Full text index는 단어 일치 순서, 단어가 얼마나 가까운지 등의 측면에서 훨씬 더 flexibility 제공 할 수 있다.

**Stemming:**

Full text search는 단어 stemming이 가능하다. 예를 들어, "run"을 검색하면 "ran" 또는 "running"에 대한 결과를 얻을 수 있다. 
대부분의 full text 엔진에는 다양한 언어로된 stem dictionaries가 존재 한다.

**Weighted Results:**

Full text index는 여러 컬럼을 포함 할 수 있다. 예를 들어 "peach pie"를 검색하면 index에 제목, 본문, 키워드 등이 포함될 수 있다.
제목과 일치한다고 검색된 결과가 패턴과 관련성이 높을수록 가중치가 높을 수 있으며, 위쪽에 표시되도록 정렬 할 수 있다.


full text searching의 약점은 다음과 같다.

**Disadvantages:**

Full text index는 표준 B-TREE 인덱스보다 몇 배 더 클 수 있다. 이러한 이유로 데이터베이스 인스턴스를 제공하는 많은 호스팅 제공 업체는
이 기능을 비활성화하거나 추가 요금을 청구한다. 예를 들어, Windows Azure에서는 full text query를 지원하지 않았다.
Full text index는 업데이트 속도가 느릴 수 있다. 데이터가 많이 변경되면 표준 인덱스와 비교하여 지연 업데이트 인덱스가 있을 수 있다.

정리하자면,

Full text search는 표준 인덱스를 사용하는 것보다 검색 속도가 매우 빠르고, 키워드와 일치하는 레코드의 유사성(similarity)을 측정할 수 있고, 
tokenization(구조화 되지 않은 텍스트(row)를 terms, phrases, tokens으로 나눔)이 가능하며,
단어의 변형(variation)을 하나의 인덱스로 축소(예를 들어 "mice" and "mouse" 또는 "electrification" and "electric"를 같은 단어로 취급) 할 수 있다.

Full text search의 단점은,
prefix search만 가능('keyword%' 검색만 가능, 예를 들어 '사과%'를 찾을때 '사과 바나나'는 검색되지만, '풋사과'는 검색되지 않음), text, binary char, varchar 타입만 Full text search가 가능하며, 한글검색은 utf-8만 가능 하고, 표준 B-TREE 인덱스 보다 몇 배는 더 크며, 텍스트 양이 많을 수록 커지는 속도가 더 빠르다. 또한 인덱스를 수정할 때도 오래 걸린다는 단점이 있다.  


다른 답변이 더 궁금하다면 `Full Text Search vs LIKE` 또는 `Full text search vs standard database search` 키워드로 구글링해보자!

### 참고
* MySQL Full Text Search Index 사용하기 (https://interconnection.tistory.com/95)
* MySQL 풀 텍스트(FULLTEXT) 검색하기 (https://kmongcom.wordpress.com/2014/03/28/mysql-%ED%92%80-%ED%85%8D%EC%8A%A4%ED%8A%B8fulltext-%EA%B2%80%EC%83%89%ED%95%98%EA%B8%B0/)
* MYSQL FULLTEXT SEARCH (https://medium.com/@pakss328/mysql-fulltext-search-eb64d489cac5)
* Fulltext search vs standard database search (https://stackoverflow.com/questions/17796717/fulltext-search-vs-standard-database-search/17796826#17796826)
* What is Full Text Search vs LIKE (https://stackoverflow.com/questions/224714/what-is-full-text-search-vs-like)
