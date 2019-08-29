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

바로 오늘 내가 정리 할 Full Text search index 이다.

Full text index 동작은 이 블로그 글(https://interconnection.tistory.com/95) 에서 잘 설명하였으니 생략하고..
또한, Full text search 사용 방법은 이 [블로그](https://kmongcom.wordpress.com/2014/03/28/mysql-%ED%92%80-%ED%85%8D%EC%8A%A4%ED%8A%B8fulltext-%EA%B2%80%EC%83%89%ED%95%98%EA%B8%B0/) 에서 잘 설명해주었으니 생략한다.

그러면 내가 진짜로 정리 할 부분은...!!
바로 바로 `Full Text Search와 LIKE의 차이` 이다!!

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

**Disadvantages:**

Full text index는 표준 B-TREE 인덱스보다 몇 배 더 클 수 있다. 이러한 이유로 데이터베이스 인스턴스를 제공하는 많은 호스팅 제공 업체는
이 기능을 비활성화하거나 추가 요금을 청구한다. 예를 들어, Windows Azure에서는 full text query를 지원하지 않았다.
Full text index는 업데이트 속도가 느릴 수 있다. 데이터가 많이 변경되면 표준 인덱스와 비교하여 지연 업데이트 인덱스가 있을 수 있다.

다른 답변이 더 궁금하다면 `Full Text Search vs LIKE` 또는 `Full text search vs standard database search` 키워드로 구글링해보자!
