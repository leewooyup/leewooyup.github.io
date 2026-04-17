---
title: "[Elasticsearch] es 검색 설계: 매핑 구조가 스코어링을 결정하는 방식"
date: 2026-04-17 11:37 +0900
lastmod: 2026-04-17 11:37 +0900
categories: Elasticsearch
tags:
  [
    mapping,
    score,
    BM25,
    TF-IDF,
    bool,
    token
  ]
---

## 🍝 match 쿼리와 text 타입

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 text 타입에서만 match 쿼리를 사용해야 한다.
</div>

`match` 쿼리는 <span style="font-size:1.12rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 검색 키워드가 포함된 모든 도큐먼트를 조회한다.</span>

```
🥊 text 타입 이외의 모든 타입은 어떻게 검색해야 할까?

term 쿼리로 검색해야 한다.
특정값과 '정확하게 일치하는 도큐먼트'를 조회할 때 사용한다.
```

🍍 es의 데이터 타입  
- 숫자  
  - 10억 이하의 정수: `integer`  
  - 10억 초과 정수: `long`  
  - 실수: `double`  
- 문자  
  - 문자열을 토큰으로 쪼개서 저장 [for 유연한 검색] : `text`  
  - 문자열 그대로 저장 [for 정확한 검색] : `keyword`  
    - ex) 휴대폰번호, 이메일, 주문번호, (계산에서 쓰는값이 아닌) 숫자로 구성된 문자  
- 불리언 `boolean`  
- 날짜 `date`  

이러한 데이터 타입을 가지고 도큐먼트의 <span style="font-size:1.12rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 각 필드가 어떤 데이터 타입을 가지고 있는지 정의하는 설정</span>을 매핑 `mapping` 이라고 한다.  

```
🎃 es 매핑의 특징

- null 허용
매핑을 정의하더라도 null이 들어와도 괜찮다.
심지어, 해당 필드가 없어도 괜찮다.

- Array 허용
배열을 위한 매핑 타입이 없다
그래서, '별도의 매핑 설정없이' 배열 형태의 데이터를 삽입할 수 있다.

```

```sql
"mappings": {
    "properties": {
        "title": {
            "type": "text"
        },
        "hashtags": {
            "type": "text"
        },
        "content": {
            "type": "text"
        }
    }
}

-- 일부 필드가 없어도 도큐먼트를 삽입할 수 있다
POST /boards/_doc
{
    "content": "세계에는 나라마다 다양한 전통과자가 있습니다."
}

-- 배열 형태의 데이터를 삽입할 수 있다.
-- 배열에 있는 값이 각각의 토큰으로 분리되어 역인덱스로 저장된다.
-- hashtags 필드에 '여행'으로 검색하면 '여행', '과자' 모두 조회된다.
POST /boards/_doc
{
    "title": "세계의 과자",
    "hashtags": ["여행", "과자"],
    "content": "세계에는 나라마다 다양한 전통과자가 있습니다."
}


```

## 🦙 복합쿼리, bool 쿼리

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 여러 조건을 조합해서 검색할 때 쓰는 기본 복합 쿼리이다.
</div>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🧶 bool 쿼리 구성</span>

```sql
{
    "bool": {
        "must": [],     -- 반드시 매칭
        "should": [],   -- 매칭되면 점수 증가
        "filter": [],   -- 점수에 영향 없는 필터
        "must_not": []  -- 제외할 조건
    }
}
```

`filter`와 `must`는 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">SQL문의 AND 조건</span>과 같은 역할을 한다고 보면 된다.  

🌩️ score에 영향을 주는 쿼리인지 아닌지로 구분해서 사용해야 한다.  
- score와 상관있는 쿼리인 경우 must를 써야 한다.  
- score와 상관없는 쿼리인 경우 filter를 써야 한다.  

```
🍫 score에 영향을 주는 대표적인 쿼리가 match 쿼리이다.
match 쿼리에는 text 타입밖에 들어오지 못하기 때문에,
text 타입을 검색하는 match 쿼리는 무조건 must 쿼리에서 사용하면 된다.
즉, (정확하게 일치하지 않아도) 관련성 있는 데이터까지 검색하는 쿼리는 must 쿼리를 사용하면 된다.

그 외에는 전부 filter 쿼리를 사용하면 된다.
term 쿼리는 어차피 정확히 일치하지 않으면 도큐먼트를 조회해오지 않기 때문에 score를 사용할 필요가 없다.
즉, 정확하게 일치하는 데이터만 검색하는 쿼리는 filter 쿼리를 사용하면 된다.

** must > match (text 타입)
** filter > term (그외 타입)
```

🍍 score  
: 검색을 할 때 관련도가 얼마나 높은지 나타내는 수치이다.  

`should` 쿼리에는 조건을 만족하면 좋고, 아님 말아도 되는 쿼리를 사용하면 된다.  
  - 가산점 부여, 상위 노출  
`must_not`: <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">SQL문의 NOT 조건</span>과 같은 역할을 한다.  

## 🍝 BM25 스코어링

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 검색어와 문서의 연관도를 정량적으로 계산하기 위한 수식을 말한다.
</div>

`elasticsearch`의 스코어링 쿼리는 내부적으로 `BM25, Best Matching 25`라는 알고리즘을 기반으로 고전적인 TF-IDF 방식을 개선한 버전이다.  

`BM25`는 이론적으로 그냥 하나의 수식인데, 이걸 실제 코드로 구현한 버전이 `Apache Lucene`이다.  

`elasticsearch`는 `Lucene` 구현 그대로를 사용한다.  
<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">BM25 스코어링</span> == <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">Lucene이 계산한 점수</span>라고 보면 된다.  

`Lucene` 기반이면 검색을 거의 `O(1)`에 가깝게 처리해서 빠르고,  
`elasticsearch` 레벨에서 `analyzer`, <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">BM25 파라미터</span>, <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">query 조합</span> 등을 설정만으로 검색 동작을 유연하게 변경할 수 있다.  

- TF `Term Frequency`  
: 단어빈도  
문서 내 검색 키워드가 얼마나 자주 등장하는가 (자주 등장할수록 점수 높음)  
단, 너무 자주 등장해도 점수에 한계는 있다 (TF가 계속 증가해도 log scale로 완화됨.)

- IDF `Inverse Document Frequency`
: 역문서 빈도  
전체 문서 중 검색 키워드가 얼마나 희귀한가 (희귀할수록 점수 높음)

- `Field Length Normalization`
: 필드 길이 보정  
짧은 문서일수록 키워드 비중이 크다고 본다 (짧을수록 점수 높음)  
즉, title 필드값의 키워드 일치가 content 필드값의 키워드 일치보다 높은 관련성이 있다고 본다.  

🌩️ bool 쿼리 내부에 여러 match 쿼리를 사용할 때, `score scale`을 고려해야 유지보수가 쉬워진다.  
- 각 match 쿼리는 독립적으로 BM25 계산한다.  
- bool 쿼리는 각 match 쿼리에서 계산된 score를 합산하여 최종 score를 구성한다.  

```
🍫 같은 BM25 점수라도 필드마다 점수의 크기 분포가 다르게 나올 수 있다.
title 필드에서,
iphone 15 pro max는 TF 높고,
길이가 content에 비해 짧으므로
score가 높게 나온다 (ex. 5.0)

content 필드에서,
iphone 15 pro max는 TF가 비슷하거나 더 높을 수는 있어도,
Field Length Normalization 때문에 희석되어
점수가 상재적으로 낮게 나올 수 있다. (ex. 0.8)

** 문제는 bool 쿼리에서 이 점수를 합치면
title score(5.0) + content score(0.8)
title score가 거의 점수를 지배하여 과도한 영향력을 가지게 된다.
```

즉, `score scale`이 다르다는 건 필드마다 BM25 점수 크기 분포가 달라 합산했을 때  
<span style="font-size:1.12rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 한쪽 필드가 과도하게 영향력을 갖게 되는 현상</span>을 말한다.  

이를 해결하기 위해  
boost 설계를 할 수 있지만, 직관적이기 보다는 boost를 감으로 조정해야 한다.  
또한, 데이터가 바뀌면 다시 튜닝이 필요하다.  

`combined_fields`는 여러 필드를 하나의 텍스트 필드로 합쳐 하나의 BM25 스코어로 계산한다.  
즉, title, content 필드가 있을 때 하나의 통합 벡터 공간에서 TF-IDF를 계산하는 방식으로  
전체 필드에서의 relevance로 표현된다.  

```
🥊 multi_match + cross_fields vs combined_fields

multi_match의 cross_fields type은 여러 필드를 하나의 가상 단일 필드처럼 취급하지만
내부적으로는 토큰(term) 기준으로 필드를 가로질러 매칭한다.
여러 필드에 대해 각각 따로 검색한뒤 결과를 합치는 방식이다.
즉, phrase 검색보다는 term 조합이라고 볼 수 있다.

{
  "multi_match": {
    "query": "iphone 15",
    "fields": ["title", "description"]
  }
}

combined_fields는 여러 필드를 진짜 하나의 텍스트 필드처럼 합친다.
때문에 TF-IDF 계산도 합쳐진 문서 기준으로 실행한다.
즉, phrase 검색도 더 자연스럽게 반영할 수 있다.

{
  "combined_fields": {
    "query": "iphone 15",
    "fields": ["title", "description"]
  }
}

** multi_match + cross_fields → term 중심 scoring
** combined_fields → document 중심 scoring

즉, 검색창 자동완성같은 키워드 매칭이 중요한 경우 cross_fields를 사용하고
문서 단위 내에서의 연관성 기준으로 score의 왜곡을 줄이고 싶을 때는 combined_fields를 사용하면 된다.
ex) 상품 검색 (title + brand + description)
```

## 🧃 게시판 인덱스 구조 설계

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🧃 스키마는 변경이 어려우니, 잘못 설계하면 재색인 해야한다.
</div>

**[1]** 필드마다 검색 vs 필터 vs 정렬 중 무엇에 쓰이는지를 먼저 정의해야 한다.

**[2]** 검색용 필드는 데이터 타입을 `text`로 설정하고, 필터/정렬/집계용 필드는 데이터 타입을 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">keyword 및 나머지 타입</span>으로 설정한다.  
<span style="font-size:1.12rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 검색용 필드는 문장을 쪼개고(토큰화), 가공해서 검색가능한 형태로 바뀐다.</span>  

**[3]** 검색용 필드 중, 한글이 값으로 들어갈 수 있는 필드에는 `nori analyzer`를 선택한다. [형태소 분석]  

**[4]** 불필요한 필드는 최소화한다.  
`text` 필드가 많으면 검색 성능에 영향이 있을 수 있다.  

```sql
PUT /boards
{
    "mappings": {
        "properties": {
            "board_id": {
                "type": "long"
            },
            "title": { -- 검색
                "type": "text",
                "analyzer": "nori"
            },
            "category": { -- 필터
                "type": "keyword"
            }, 
            "is_notice": { -- 필터
                "type": "boolean"
            },
            "view_count": { -- 필터 or 정렬
                "type": "long"
            },
            "created_at": { -- 필터 or 정렬
                "type": "date"
            }
        }
    }
}

-- 도큐먼트 삽입
POST /boards/_doc
{
    "board_id": 1,
    "title": "공지사항 서비스 점검 및 서버 안정화 안내",
    "content": "서비스 안정화를 위한 정기 점검이 진행됩니다. 일부 기능이 제한될 수 있습니다.",
    "category": "notice",
    "is_notice": true,
    "view_count": 1520,
    "created_at": "2026-04-06T10:05:00"
}

POST /boards/_doc
{
    "board_id": 2,
    "title": "회원가입 오류 발생 시 해결 방법",
    "content": "회원가입 실패 시 이메일 형식, 인증번호, 비밀번호 조건을 확인해주세요.",
    "category": "faq",
    "is_notice": false,
    "view_count": 860,
    "created_at": "2026-04-07T22:00:00"
}

POST /boards/_doc
{
    "board_id": 3,
    "title": "이벤트 참여 방법 및 당첨자 발표 일정",
    "content": "이벤트 참여는 로그인 후 가능하며, 당첨자는 공지사항에서 확인할 수 있습니다.",
    "category": "event",
    "is_notice": false,
    "view_count": 430,
    "created_at": "2026-04-08T11:15:00"
}

POST /boards/_doc
{
    "board_id": 4,
    "title": "시스템 긴급 장애 대응 공지",
    "content": "예기치 않은 장애로 인해 시스템 복구 작업이 진행 중입니다.",
    "category": "notice",
    "is_notice": true,
    "view_count": 2100,
    "created_at": "2026-04-09T14:30:00"
}

POST /boards/_doc
{
    "board_id": 5,
    "title": "비밀번호 변경 및 보안 강화 가이드",
    "content": "보안을 위해 주기적인 비밀번호 변경과 2단계 인증 설정을 권장합니다.",
    "category": "guide",
    "is_notice": false,
    "view_count": 670,
    "created_at": "2026-04-10T09:00:00"
}

POST /boards/_doc
{ 
  "board_id": 6,
  "title": "Login 인증 실패 및 접속 오류 해결",
  "content": "Login 실패 시 캐시 삭제, 비밀번호 재설정, 네트워크 상태를 확인해주세요.",
  "category": "guide",
  "is_notice": false,
  "created_at": "2026-04-05T13:40:00",
  "view_count": 540
}
```

---

```sql
-- ✨ bool 쿼리 사용하기 (filter와 must 구분해서)
-- '비밀번호' 키워드와 연관된 title이면서,
-- category가 'guide'과 정확히 일치하고, 공지글이 아닌 게시글 조회
GET /boards/_search
{
    "query": {
        "bool": {
            "must": [
                {
                    "match": {
                        "title": "비밀번호"
                    }
                }
            ],
            "filter": [
                {
                    "term": {
                        "category": "guide"
                    }
                },
                {
                    "term": {
                        "is_notice": false
                    }
                }
            ]
        }
    }
}

-- ✨ bool 쿼리 사용하기 (must_not 써보기)
-- '비밀번호' 키워드와 연관된 title이면서,
-- category가 'event'가 아니고, 공지글이 아닌 게시글 조회
GET /boards/_search
{
    "query": {
        "bool": {
            "must_not": [
                {
                    "term": {
                        "category": "event"
                    }
                }
            ],
            "filter": [
                {
                    "term": {
                        "is_notice": false
                    }
                }
            ],
            "must": [
                {
                    "match": {
                        "title": "검색엔진"
                    }
                }
            ]
        }
    }
}

-- 다르게 표현할 수도 있다
GET /boards/_search
{
    "query": {
        "bool": {
            "must_not": [
                {
                    "term": {
                        "category": "event"
                    }
                },
                {
                    "term": {
                        "is_notice": true
                    }
                }
            ],
            "must": [
                {
                    "match": {
                        "title": "비밀번호"
                    }
                }
            ]
        }
    }
}
```

---

✨ 숫자/날짜값에 대해 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">범위조건으로 데이터를 조회</span>하고 싶을 때 - `range`  
- `gte` : 이상
- `lte` : 이하
- `gt` : 초과
- `lt` : 미만  

```sql
-- ✨ range 쿼리 사용하기 (범위 조건)
-- 조회수가 1,000 이상이면서,
-- 게시글 등록 날짜가 2026년 04월 08일 이후인 게시글 조회
GET /boards/_search
{
    "query": {
        "bool": {
            "filter": [
                {
                    "range": {
                        "view_count": {
                            "gte": 1000
                        }
                    }
                },
                {
                    "range": {
                        "created_at": {
                            "gte": "2026-04-08"
                        }
                    }
                }
            ]
        }
    }
}
```

---

✨ 오타가 있더라도 유사한 단어를 포함한 데이터를 조회하고 싶을 때 - `fuzziness`  
: `"fuzziness": "AUTO"` : 단어 길이에 따른 오타 허용 개수를 자동으로 설정  

```sql
GET /boards/_search
{
    "query": {
        "match": {
            "title": {
                "query": "logn",
                "fuzziness": "AUTO"
            }
        }
    }
}
```

---

✨ 여러 필드에서 검색 키워드가 포함된 데이터를 조회하고 싶을 때 - `multi_match`  
: 하나의 필드에서는 match를 쓰면 되지만, 두개 이상의 필드에서는 `multi_match`를 써야한다.  
: 찾고자 하는 필드를 배열로 여러개 지정할 수 있다.  
: 배열에 나열된 필드 중 하나의 필드값에만 검색키워드가 들어가있어도 조회된다.

```sql
GET /boards/_search
{
    "query": {
        "multi_match": {
            "query": "비밀번호 재설정 주기",
            "fields": ["title^2", "content"] -- 특정 필드 가중치 설정 가능
        }
    }
}

```

---

✨ 검색한 키워드를 하이라이팅 처리하고 싶을 때 - `highlight`  

```sql
GET /boards/_search
{
    "query": {
        "multi_match": {
            "query": "비밀번호 변경 주기",
            "fields": ["title^2", "content"]
        }
    },
    "highlight": {
        "fields": {
            "title": {
                "pre_tags": ["<mark>"]
                "post_tags": ["</mark>"]
            },
            "content": {
                "pre_tags": ["<b>"],
                "post_tags": ["</b>"]
            }
        }
    }
}
```

## 🗿 페이지네이션

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #462679;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🗿 시스템의 과부하를 방지하기 위해, 데이터 조회시 고려해야하는 옵션이다
</div>

`size`와 `from`을 활용하면 페이지네이션 `Pagenation`의 기능처럼 데이터를 불러올 수 있다.  

```sql
GET /boards/_search
{
    "query": {
        "match": {
            "title": "방법"
        }
    },
    "size": 2, -- 하나의 page 당 도큐먼트 개수
    "from": 0 -- offset (0번째 도큐먼트부터, 1 page)
}

GET /boards/_search
{
    "query": {
        "match": {
            "title": "방법"
        }
    },
    "size": 2,
    "from": 2, -- (2 page)
}

-- ✨ 정렬
GET /boards/_search
{
    "query": {
        "match": {
            "title": "방법"
        }
    },
    "sort": [
        {
            "view_count": {
                "order": "desc"
            }
        }
    ]
}
```

## 🗿 멀티 필드

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #462679;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🗿 하나의 필드에 여러 개의 타입을 선언할 수 있다.
</div>

하나의 필드에 대해서 유연한 검색`match` 과 정확한 검색`term` 둘 다 하고 싶은 경우에 사용한다.  

```sql
PUT /products
{
    "mappings": {
        "properties": {
            "name": { -- 검색
                "type": "text",
                "analyzer": "nori"
            },
            "category": { -- 검색 or 필터
                "type": "text",
                "analyzer": "nori",
                "fields": { -- 멀티 필드
                    "raw": { -- 사용자 정의값
                        "type": "keyword"
                    }
                }
            }
        }
    }
}

POST /products/_bulk
{ "index": {} }
{ "name": "삼성 무선 청소기 Jet 90", "category": "가전 청소기 생활가전" }

{ "index": {} }
{ "name": "LG 디오스 냉장고 오브제컬렉션", "category": "가전 냉장고 대형가전" }

{ "index": {} }
{ "name": "애플 iPhone 15 Pro Max", "category": "스마트폰 휴대폰 모바일기기" }

{ "index": {} }
{ "name": "나이키 러닝화 Air Zoom Pegasus", "category": "운동화 신발 스포츠용품" }

{ "index": {} }
{ "name": "다이슨 에어랩 스타일러", "category": "헤어드라이어 뷰티 가전" }
```

위와 같이 멀티 필드를 사용하면 해당 필드에 데이터를 삽입할 때  
`text` 타입의 형태로 데이터를 나눠서 저장하고,  
`keyword` 타입의 형태로도 데이터를 저장한다.  

저장된 토큰 확인
```sql
GET /products/_analyze
{
    "field": "category",
    "text": "가전 청소기 생활 가전"
}

GET /products/_analyze
{
    "field": "category.raw",
    "text": "가전 청소기 생활 가전"
}
```

## 🗿 자동완성

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #462679;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🗿 검색 키워드의 일부를 입력했을 때, 검색어를 추천해 주는 기능을 말한다.
</div>

`🎃 search_as_you_type`  
: `elasticsearch`에서 자동완성 기능을 쉽게 구현할 수 있게 <span style="font-size:1.12rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 설계된 데이터 타입</span>이다.  
: `text` 타입처럼 `analyzer`를 거쳐 토큰으로 분리된다.  
: 내부적으로 멀티 필드를 만들어 관리한다.  
- `_2gram` : 두단어씩 묶어서 토큰을 만든다.  
- `_3gram` : 세단어씩 묶어서 토큰을 만든다.  

```sql
PUT /products
{
    "mappings": {
        "properties": {
            "name": {
                "type": "search_as_you_type",
                "analyzer": "nori"
            }
        }
    }
}

-- ✨ 토큰 확인하기
-- products 인덱스의 name이란 필드에 적용된 analyzer를 이용해서 text를 분석
GET /products/_analyze
{
    "field": "name",
    "text": "삼성 무선 청소기 Jet 90"
}

GET /products/_analyze
{
    "field": "name._2gram",
    "text": "삼성 무선 청소기 Jet 90"
}

GET /products/_analyze
{
    "field": "name._3gram",
    "text": "삼성 무선 청소기 Jet 90"
}

-- ✨ 자동완성 쿼리
GET /products/_search
{
    "query": {
        "multi_match": { -- 여러 필드에 걸쳐서 키워드 검색
            "query": "",
            "type": "bool_prefix",
            "field": [
                "name",
                "name._2gram",
                "name._3gram"
            ]
        }
    }
}
```

🌩️ `bool_prefix`  
: 앞쪽 단어는 `match`, 마지막 단어는 `prefix match`  
'삼성 무선 청소'라고 검색하면 '삼성'과 '무선'은 역인덱스에 저장된 토큰과 일치하는 데이터를 찾고,  
마지막 단어인 '청소'는 역인덱스에 저장된 토큰 중에 '청소'로 시작하는 데이터를 찾는다.  
&nbsp;  
: 연속적으로 단어가 일치할 수록 score를 더 높게 측정해 상위노출되게 하기 위해  
fields에 `._2gram`과 `._3gram`을 추가  
&nbsp;  
자동완성 기능을 위해서 한글자씩 칠때마다 `elasticsearch`로 요청을 보내야 한다.