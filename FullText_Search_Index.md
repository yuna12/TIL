# MySQL Full Text Search Index


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
FullTEXT는 TEXT, BLOB, VARCHAR 등 가변적이고 일반적인 Index의 효율성이 떨어지는 부분에서 많은 효과를 가져올 수 있는 인덱스이다.
FULLTEXT Index는 텍스트 필드에 '%검색문자열%'과 비슷한 형태의 검색 결과를 얻을 수 있고 TEXT에 최적화된 Index 방식이다.
```

바로 오늘 내가 정리 할 Full Text search index 이다.

Full text index 동작은 이 블로그 글(https://interconnection.tistory.com/95) 에서 잘 설명하였으니 생략하고..
또한, Full text search 사용 방법은 이 [블로그](https://kmongcom.wordpress.com/2014/03/28/mysql-%ED%92%80-%ED%85%8D%EC%8A%A4%ED%8A%B8fulltext-%EA%B2%80%EC%83%89%ED%95%98%EA%B8%B0/) 에서 잘 설명해주었으니 생략한다.

그러면 내가 진짜로 정리 할 부분은...!!
바로 바로 `Full Text Search와 LIKE의 차이` 이다!!

이쯤되니 궁금해지지 않겠는가.. % 연산자를 이용해서 LIKE로 검색하는 것과 Full Text Search로 검색하는 것의 차이는 무엇일까

마찬가지로 구글링해보니 스택오버플로우에 [나와 같은 질문](https://stackoverflow.com/questions/224714/what-is-full-text-search-vs-like)을 한 사람이 있었다.

다음은 가장 많은 투표를 받은 답변을 정리하였다.

Full text search와 LIKE 연산자의 차이는 `"precision(정확도)"과 "recall(재현율)"의 tradeoff` 라고 했다.
LIKE operator는 100% precision을 얻을 수 있는 반면에, Full text search는 더 높은 recall을 위해 precision을 조정할 수 있는 유연성이 있다.

Full text search는 인덱스를 사용해서 구현 된다. 이 인덱스의 키는 개별 용어(terms)이며, 값은 해당 용어를 포함하고 있는 레코드 세트이다.
Full text search는 이러한 레코드 세트의 intersection, union 등을 계산하도록 최적화되며 일반적으로 주어진 레코드가 검색 키워드와 얼마나 많이 일치하는지를 정려화하는 순위 알고리즘을 제공한다.

반면, LIKE 연산자를 인덱싱 되지 않은 컬럼에 적용하면, 매칭되는 패턴을 찾기 위해 full scan이 일어나 매우 비효율적이며,
인덱싱된 컬럼을 사용한다 해도, 대부분의 인덱스 조회 보다 효율성이 훨씬 낮다.

또 다른 특성은 내일 이어서 정리하겠다.
