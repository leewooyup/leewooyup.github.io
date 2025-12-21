---
title: "[LangChain] 외부 기능을 LLM이 필요시 사용할 수 있게 하는 ToolCalling과 사용자 정의 도구"
date: 2025-12-21 13:05 +0900
lastmod: 2025-12-21 13:05 +0900
categories: LangChain
tags:
  [
	TavilySearch,
	CSE,
	Crawling,
	content,
	tool_calls,
	ToolMessage,
	as_tool,
	RunnableLambda,
	few-shot
  ]
---

## 🪵 웹 검색 도구 - TavilySearch

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 AI기반의 웹 검색 API를 제공하는 서비스이다
</div>

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(월 1000번까지 무료 사용 가능)</span>  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">🗝️ 인증키: TAVILY_API_KEY 설정</span>

```python
from langchain_community.tools import TavilySearchResults

query = "스테이크와 어울리는 와인을 추천해주세요"

# 1] 도구 정의
# Tavily 검색 도구 초기화 (최대 2개의 결과를 반환)
web_search = TavilySearchResults(max_results=2)

# 웹 검색 실행
search_results = web_search.invoke(query)

from result in search_results:
    print(result)
    print("-"*100)
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
{
    'url': 'https://mashija.com/....,
    'content': '# 스테이크와 어울리는 최고의 와인: 무엇을 고를 것인가?\n\n카베르네 소비뇽(Cabernet Sauvignon) 및 말벡(Malbec)과 같은 전형적인 선택부터 더 가벼운 레드 와인, 심지어 화이트 와인과 맛있는 스테이크를 페어링하는 방법까지, 우리의 아카이브에서 가져온 최고의 조언과 최근 디캔터 ...
}
----------------------------------------------------------------------------------------------------
{
    'url': 'https://alcohol.hobby-tech.com/entry/...',
    'content': '• 4. 메를로 (Merlot) – 부드럽고 균형 잡힌 와인\n\n• 5. 진판델 (Zinfandel) – 달콤한 향과 강한 바디감\n\n• 어떤 와인을 선택해야 할까?\n\n스테이크를 제대로 즐기려면 와인 선택이 중요합니다. 고기의 종류와 조리법에 따라 어울리는 와인이 달라지는데요. 어떤 와인을 선택해야 풍미를 극대화할 수 있을까요? 오늘은 스테이크와 가장 잘 어울리는 와인 5가지를...'
}
----------------------------------------------------------------------------------------------------
```

<div style="margin-bottom:15px;font-size:15px;background-color:#F7F7F7;border-radius:5px;padding:15px;color:#34343c;">
<span style="font-weight:bold;">🥊 Google Custom Search API vs TavilySearch</span><br>
간단히 말하면 Google CSE는 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">링크 중심 검색</span>이고,<br>
Tavily는 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">내용 중심 검색</span>이다.<br><br>
Google CSE는 요약정보인 snippet을 제공해주긴 하지만, 사실상 실무에서 사용하기엔 정보의 내용이 짧고 불완전하다.<br>
실제 핵심 내용은 link를 타고 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">직접 크롤링해야 한다.</span><br>
HTML 파싱, 광고 제거, 본문 추출을 직접 구현해야 한다.<br><br>
Tavily는 본문 텍스트를 추출해서 제공한다.<br>
LLM 프롬프트에 넣기 좋은 길이로 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">검색 품질이 AI에 최적화되어 있다.</span><br><br>
그렇다고 마냥 Tavily가 좋다기보다는,<br>
Google CSE는 <span style="padding:0 6px;font-size:16px;border-radius:5px;background-color:#DEDEDE;color:#5F5F5F;">검색 대상 지정 및 세부 제어 요소가 많다.</span><br>
즉 정확한 사이트 통제가 필요하다면 Google CSE가 유리하다.
</div>

---

## 🦙 도구 호출 (Tool Calling)

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 랭체인에서 LLM이 외부기능이나 API를 사용할 수 있게하는 매커니즘을 말한다.
</div>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">⚔️ 도구 (Tool) 의 구성요소</span>  

```
name: 도구의 이름
description: 도구가 수행하는 작업에 대한 설명
JSON schema: 도구의 입력을 정의하는 스키마
function: 실행할 함수 (비동기 함수도 가능)

⚠️ LLM이 'name'과 'description'을 보고 어떤 역할을 하는지 파악한다.
```

`LLM`을 이용해서 `ToolCalling`을 하게 되면 사용자의 질문으로 그대로 이용해서 검색하는 것이 아니라  
`LLM`이 <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 적절한 검색어로 변환해서 검색을 수행한다.</span>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🗿 도구 호출의 동작 방식</span>

```
1] 도구 정의
	1-test] 도구 직접 호출
2] LLM 모델에 도구를 바인딩
	2-test] 도구가 바인딩된 LLM 직접 호출
3] 프롬프트 설계
4] LLM 체인 정의
5] 도구 실행 체인 정의
	5-1] LLM 체인 실행을 통한 도구 호출
	5-2] 도구 호출의 결과(ToolMessage)를 받아 최종 답변 생성 (LLM 체인 실행)
6] 문자열 포맷팅
```

---

## 🦙 LLM 모델에 도구를 바인딩

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">bind_tools</span>을 하면 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Structured Tool</span> (구조화된 도구)로 변환해서 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">LLM</span>에 바인딩해준다
</div>

사용자의 입력이 들어왔고, `LLM`이 도구 호출이 필요하다고 판단했을 때  
사용할 수 있는 도구를 가지고 있다면,  
`LLM`이 도구에서 <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 정의한 스키마에 맞게 데이터를 변환한다.</span> <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(Dictionary 형태)</span>

즉, 도구 (함수, API) 에서 <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 미리 정의해둔 스키마 구조에 맞게 데이터를 변환하는 작업(구조화)</span> 을 `ToolCalling`이라 한다.

```python
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini 모델 초기화
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

# 2] LLM 모델에 (웹 검색)도구를 바인딩
llm_with_tools = llm.bind_tools(tools=[web_search])

# 도구 호출이 필요없는 쿼리로, LLM 호출을 수행
query = "안녕하세요."
ai_msg = llm_with_tools.invoke(query)

# LLM의 전체 결과 출력
pprint(ai_msg)
```

🍍 `pprint (pretty-print)`  
: 자료구조를 사람이 읽기 좋게 여러줄로 정렬해서 출력

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
AIMessage(
	content='안녕하세요! 무엇을 도와드릴까요?\n', 
	additional_kwargs={}, 
	response_metadata={
		'prompt_feedback': {
			'block_reason': 0, 
			'safety_ratings': []
		}, 
		'finish_reason': 'STOP', 
		'safety_ratings': []
	}, 
	id='run--b6d291dc-49be-4843-9dce-1256bdbcf3a3-0',
	usage_metadata={
		'input_tokens': 60,
		'output_tokens': 15,
		'total_tokens': 75
	}
)
```
`LLM`이 해당 쿼리는 도구 호출이 필요하지 않다고 판단해서  
그 결과로 `content`에 응답을 바로 생성했다.  
도구호출을 하지 않았기 때문에 `additional_kwargs`가 빈객체로 반환  
&nbsp;  
만약 도구 호출을 하게끔 쿼리를 넣는다면,

```python
query = "스테이크와 어울리는 와인을 Tavily 웹 검색을 이용해서 추천해주세요."
ai_msg = llm_with_tools.invoke(query)

pprint(ai_msg)
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
AIMessage(
	content='',
	additional_kwargs={
		'function_call': {
			'name': 'tavily_search_results_json',
			'arguments': '{
				"query": "\\uc2a4\\ud14c\\uc774\\ud06c\\uc640 \\uc5b4\\uc6b8\\ub9ac\\ub294 \\uc640\\uc778 \\ucd94\\ucc9c"
			}'
		}
	}, 
	response_metadata={
		'prompt_feedback': {
			'block_reason': 0,
			'safety_ratings': []
		}, 
		'finish_reason': 'STOP',
		'safety_ratings': []
	},
	id='run--f708eead-1201-4726-a9c6-de0781b0578c-0',
	tool_calls=[
		{
			'name': 'tavily_search_results_json',
			'args': {
				'query': '스테이크와 어울리는 와인 추천'
			}, 
			'id': '07ea418b-46a4-47b2-8120-d58c88ec2597',
			'type': 'tool_call'
		}
	],
	usage_metadata={
		'input_tokens': 79,
		'output_tokens': 20,
		'total_tokens': 99
	}
)
```

`content` 키값이 빈문자열이라는 것은 모델이 응답을 생성 않았기 때문이다.  
대신, `AIMessage.additional_kwargs`에 `tavily_search_results_json`이라는 함수 호출 이력이 있고  
`AIMessage.tool_calls`에 실제 호출 정보가 담겨있다.  
&nbsp;  
`AIMessage.tool_calls.args.query`에 사용자 입력이 요약된 형태의 질문이 들어가 있는데  
이는, 도구호출 시 `LLM`이 자연어 프롬프트를 읽고 `args`에 `JSON`을 생성해야 하는데  
이 때 해당 모델이 핵심 질의만을 뽑에서 `args.query` 키값에 넣어준 결과이다.  
즉, 해당 메시지는 `LLM`이 <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 도구 호출 단계라고 판단</span>하고 `tool_calls`을 제안한 것 뿐이다.

<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">※ 도구를 실제로 실행하진 않고, 그저 검색에 이용할 쿼리와 어떤 도구를 이용할 지를 반환한 것 뿐이다.</span>

```python
tool_call = ai_msg.tool_calls[0]

# [방법1] args 스키마 사용
# 핵심질의가 들어있는 args 속성값을 전달해 웹검색을 실행한다.
tool_output = web_search.invoke(tool_call["args"])

# [방법2] - 가장 많이 사용
# 도구 그 자체를 넘겨서 웹검색을 실행하면 ToolMessage 객체를 반환한다.
# tool_output2 = web_search.invoke(tool_call)

print(f"{tool_call['name']}" 호출결과:")
print(tool_output)
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
tavily_search_results_json 호출 결과:
[
    {
        'url': 'https://mashija.com/...',
        'content': '# 스테이크와 어울리는 최고의 와인: 무엇을 고를 것인가?\n\n카베르네 소비뇽(Cabernet Sauvignon) 및 말벡(Malbec)과 같은 전형적인 선택부터 더 가벼운 레드 와인, 심지어 화이트 와인과 맛있는 스테이크를...'
    }, 
    {
        'url': 'https://alcohol.hobby-tech.com/entry/...',
        'content': '• 4. 메를로 (Merlot) – 부드럽고 균형 잡힌 와인\n\n• 5. 진판델 (Zinfandel) – 달콤한 향과 강한 바디감\n\n• 어떤 와인을 선택해야 할까?\n\n스테이크를 제대로 즐기려면 와인 선택이 중요합니다. 고기의 종류와 조리법에 따라 어울리는 와인이 달라지는데요. 어떤 와인을 선택해야 풍미를...'
    }
]
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🪄 도구 호출의 구조화된 메시지 만들기, ToolMessage</span>  

```python 
from langchain_core.messages import ToolMessage

tool_message = ToolMessage(
	content=tool_output,
	tool_call_id=tool_call["id"],
	name=tool_call["name"]
)

print(tool_message)
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
content=[
	{
		'url': 'https://mashija.com/%EC%8A%A4%ED%85%8C%EC%9D%B4%E...',
		'content': '# 스테이크와 어울리는 최고의 와인: 무엇을 고를 것인가?\n\n카베르네 소비뇽...'
	}, 
	{
		'url': 'https://m.blog.naver.com/wineislikeacat/223096696241',
		'content': '카베르네 소비뇽(Carbernet Sauvignon), 시라(Syrah) 품종을 추천드려요!...'
	}
] 
name='tavily_search_results_json'
tool_call_id='4e8d2316-fdd5-4513-9d8f-29d0bd8bf5f6'
```

---

## 🦙 ToolMessage를 LLM에 전달하여 최종답변 생성하기 

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">placeholder</span>는 대화이력 또는 도구 호출 결과를 프롬프트에 삽입하기 위한 자리 표시자이다.
</div>

`placeholder`에 도구 호출 결과 `(ToolMessage)`를 넣어 `LLM`이 참고할 수 있게 한다.

```python
from datetime import datetime
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableConfig, chain
from langchain_google_genai import ChatGoogleGenerativeAI

today = datetime.today().strftime("%Y-%m-%d")

prompt = ChatPromptTemplate([
	("system", f"You are a helpful AI assistant. Today's date is {today}."),
	("human", "{user_input}"),
	("placeholder", "{messages}"),
])

llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

# LLM에 도구(검색도구)를 바인딩
llm_with_tools = llm.bind_tools(tools=[web_search])

# LLM 체인 생성
llm_chain = prompt | llm_with_tools

# 체인 데코레이터, 도구 실행 체인 정의
@chain
def web_search_chain(user_input: str, config: RuunableConfig):
	input_ = {"user_input": user_input}
	ai_msg = llm_chain.invoke(input_, config=config)
	print("ai_msg: \n", ai_msg)
	print("-"*50)

	if ai_msg.tool_calls: # 도구 호출이 있는 경우
		tool_msgs = web_search.batch(ai_msg.tool_calls, config=config)
		print("tool_msgs: \n", tool_msgs)
	
		# ai_msg: 도구 호출 메시지 (웹검색이 필요하다고 말한)
		# tool_msgs: 실제 도구가 반환한 결과
		return llm_chain.invoke({**input_, "messages": [ai_msg, *tool_msgs]}, config=config)
	else:
		return ai_msg

# 체인 실행
response = web_search_chain.invoke("오늘 모엣샹동 샴페인의 가격은 얼마인가요?")

pprint(response.content)
```

## 🍝 사용자 정의 도구

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">@tool</span> 데코레이터를 이용하면 함수를 랭체인의 도구로 변환할 수 있다
</div>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🎃 도구 함수 작성 가이드라인</span>

```
- 명확한 입출력 정의
- 단일 책임 원칙 준수
- 도구 설명(description) 작성, docstring
  (LLM이 해당 description을 보고 도구의 기능을 이해)

※ 함수명이 도구의 'name'이 되고, docstring은 'description', 매개변수는 'args'가 된다.
```

```python
from langchain_community.tools import TavilySearchResults
from langchain_core.tools import tool

@tool
def search_web(query: str) -> str:
	"""Searches the internet for information that does not exists in the database or for the latest information"""

	tavily_saerch = TavilySearchResults(max_results=2)
	docs = tavily_search.invoke(query)
	# Dictionary로 이루어진 배열, 문자열 포맷팅 필요

	formatted_docs = "\n---\n".join([
		f'<Document href="{doc["url"]}"/>\n{doc["content"]}\n</Document>'
		for doc in docs
	])

	if len(formatted_docs) > 0:
		return formatted_docs

	return "관련 정보를 찾을 수 없습니다."

print(type(search_web))
# <class 'langchain_core.tools.structured.StructuredTool'>
print(search_web.name)
# search_web
print(search_web.description)
# ('Searches the internet for information that does not exist in the database or for the latest information.')
print(search_web.args_schema.schema())
# {'description': 'Searches the internet for information that does not exist in '
#                'the database or for the latest information.',
# 'properties': {'query': {'title': 'Query', 'type': 'string'}},
# 'required': ['query'],
# 'title': 'search_web',
# 'type': 'object'}

# 도구 직접 호출
query = "스테이크와 어울리는 와인을 추천해주세요."
search_results = search_web.invoke(query)

print(search_results)
```

<span style="font-size:.98rem;font-weight:normal;padding:3px 5px;">🗞️ 출력결과</span>

```
<Document href="https://mashija.com/..."/>
# 스테이크와 어울리는 최고의 와인: 무엇을 고를 것인가?

카베르네 소비뇽(Cabernet Sauvignon) 및 말벡(Malbec)과 같은 전형적인 선택부터 더 가벼운 레드 와인, 심지어 화이트 와인과 맛있는 스테이크를 페어링하는 방법까지, 우리의 아카이브에서 가져온 최고의 조언과 최근 디캔터 전문가가 추천한 와인을 소개한다.

<스테이크를 곁들인 레드 와인을 위한 5가지 전형적인 선택>

• 카베르네 소비뇽(Cabernet Sauvignon)  
• 말벡(Malbec)  
• 그르나슈/쉬라즈 블렌드(Grenache / Shiraz blends)  
• 시라/쉬라즈(Syrah / Shiraz)  
• 산지오베제(Sangiovese)

육즙이 풍부한 스테이크와 맛있는 와인이 있는 저녁 식사는 적어도 고기 애호가들에게 인생의 큰 즐거움일 것이다. [...] ‘멋지고 생동감 넘치는 카베르네 프랑(Cabernet Franc)은 어떤가? 아니면 카리냥(Carignan), 쌩소(Cinsault) 또는 서늘한 기후에서 생산한 시라(Syrah)는 어떨까? DWWA 칠레 지역 의장이자 Decanter Retailer Awards 회장인 리차즈...
</Document>
```

```python
# LLM에 도구를 바인딩
llm_with_tools = llm.bind_tools(tools=[search_web])

query = "스테이크와 어울리는 와인을 추천해주세요."
ai_msg = llm_with_tools.invoke(query)

pprint(ai_msg)
# AIMessage(content='', additional_kwargs={'function_call': {'name': 'search_web', ...})
pprint(ai_msg.tool_calls)
# [{'args': {'query': '스테이크와 어울리는 와인'},
#   'id': '709ccead-17bb-40b2-a7cd-501a049a7277',
#   'name': 'search_web',
#   'type': 'tool_call'}]
```

<span style="font-size:0.98rem;padding:3px 5px;background-color:#e1d1b8;border-radius:4px;color:#34343c;font-weight:normal;">🌩️ 함수 정의 후, 필요할 때만 도구로 변환하고 싶을 때는 as_tool 함수를 사용하면 된다.</span>  
- `Runnable` 객체로 변환 후, `as_tool` 메서드 사용  
- `@tool`은 함수 정의 시 바로 `Tool`로 변환하고, 도구로만 사용 가능하다.


```python
from langchain_community.document_loaders import WikipediaLoader
from langchain_core.documents import Document
from langchain_core.runnables import RunnableLambda
from pydantic import BaseModel, Field
from typing import List

def search_wiki(input_data: dict) -> List[Document]:
	"""Search Wikipedia doucments based on user input(query) and return k documnets"""
	query = input_data["query"]
	k = input.data.get("k", 2)
	# 위키피디아 문서를 검색
	wiki_loader = WikipediaLoader(query=query, load_max_docs=k, lang="ko") # default: "en"
	wiki_docs = wiki_loader.load()
	return wiki_docs

# 도구 호출에 사용할 입력 스키마 정의
class WikiSearchSchema(BaseModel):
	"""Input schema for Wekipedia search."""
	query: str = Field(..., description="The query to search for in Wikipedia")
	k: int = Field(2, description="The number of documnents to return (default is 2)")

# 위키피이다 문서 로더를 Runnable로 변환
runnable = RunnableLambda(search_wiki)

# 도구로 변환, as_tool
wiki_search = runnable.as_tool(
	name="wiki_search",
	description=dedent("""
		Use this tool when you need to search for information on Wikipedia.
        It searches for Wikipedia articles related to the user's query and returns
        a specified number of documents. This tool is useful when general knowledge
        or background information is required.
	"""),
	args_schema=WikiSearchSchema
)
# 모든 정보들이 LLM의 프롬프트로 변환되어 전달되기 때문에
# 상세하게 작성하는게 성능을 높이는데 좋다

# 1-test] (위키 검색)도구 직접 호출
query = "파스타의 유래"
wiki_results = wiki_search.invoke({"query": query})

# 검색 결과 출력
for result in wiki_results:
	print(result)
	print("-"*50)

# 2] LLM 모델에 도구를 바인딩
llm_with_tools = llm.bind_tools(tools=[wiki_search])

# 2-test] 도구가 바인딩된 LLM 직접 호출
query = "서울 강남의 유명한 파스타 맛집은 어디인가요? 그리고 파스타의 유래를 알려주세요"
ai_msg = llm_with_tools.invoke(query)

pprint(ai_msg)
print("-"*50)

pprint(ai_msg.content)
print("-"*50)

pprint(ai_msg.tool_calls)
print("-"*50)
```

## 🧃 LCEL 체인을 도구로 변환

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🧃 위키피디아 문서를 검색하고 내용을 요약하는 체인
</div>

```python
from datetime import datetime
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnableLambda, RunnableConfig, chain
from langchain_community.document_loaders import WikipediaLoader
from langchain_google_genai import ChatGoogleGenerativeAI

# 사용자 정의 도구, 평소에는 함수로 사용
def wiki_search_and_summarize(input_data: dict):
	wiki_loader = WikipediaLoader(query=input_data["query"], load_max_docs=2, lang="ko")
	wiki_docs = wiki_loader.load()

	# html 형태로 포맷팅
	formatted_docs = [
		f'<Document source="{doc.metadata["source"]}\n{doc.page_content}\n</Document>'
		for doc in wiki_docs
	]

	return formatted_docs

summary_prompt = ChatPromptTemplate.from_template(
	"Summarize the following text in a concise manner:\n\n{context}\n\nSummary:"
)

llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

# 4] (검색요약) LLM 체인 정의
summary_chain = (
	{"context": RunnableLambda(wiki_search_and_summarize)}
	| summary_prompt | llm | StrOutputParser()
)

# 체인 실행
# query 자체가 RunnableLambda에 의해서 wiki_search_and_summarize 함수로 전달
# 해당 함수의 반환값이 "context"의 값으로 들어가게 된다
summarized_text = summary_chain.invoke({"query": "파스타의 유래"})

pprint(summarized_text)

# 도구 호출에 사용할 입력 스키마 정의
class WikiSummarySchema(BaseModel):
	"""Input schema for Wikipedia search."""
	query: str = Field(..., description="The query to search for in Wikipedia")

# 체인 -> 도구로 변환
wiki_summary_tool = summary_chain.as_tool(
	name="wiki_summary",
	description=dedent("""
		Use this tool when you need to search for information on Wikipedia.
        It searches for Wikipedia articles related to the user's query and returns
        a summarized text. This tool is useful when general knowledge
        or background information is required.
	"""),
	args_schema=WikiSummarySchema
)

print(type(wiki_summary_tool))
print("-"*50)

print("name:")
print(wiki_summary_tool.name)
print("-"*50)

print("description:")
print(wiki_summary_tool.description)
print("-"*50)

print("schema:")
pprint(wiki_summary_tool.args_schema.schema())
print("-"*50)

today = datetime.today().strftime("%Y-%m-%d")

prompt = ChatPromptTemplate([
	("system", f"Your are a helpful AI assistant. Today's date is {today}."),
	("human", "{user_input}"),
	("placeholder", "{message}")
])

# LLM에 도구를 바인딩
llm_with_tools = llm.bind_tools(tools=[wiki_summary_tool])

llm_chain = prompt | llm_with_tools

# 도구 실행 체인 정의
@chain
def wiki_summary_chain(user_input: str, config: RunnableConfig):
	input_ = {"user_input": user_input}
	ai_msg = llm_chain.invoke(input_, config=config)
	print("ai_msg: \n", ai_msg)
	print("-"*50)

	tool_msgs = wiki_summary.batch(ai_msg.tool_calls, config=config)
	print("tool_msgs: \n", tool_msgs)
	print("-"*50)

	# 도구 호출 메시지와 도구를 실행한 결과 메시지를 모아 messages에 전달
	# LLM 입장에서는 질문도 확인할 수 있고, 질문에 필요한 정보도 확인할 수 있다.
	return llm_chain.invoke({**input_, "messages": [ai_msg, *tool_msgs]}, config=config)

response = wiki_summary_chain.invoke("파스타의 유래에 대해 알려주세요.")

pprint(response.content)
```

🍍 `batch`  
: 도구 호출이 여러 개인 경우

```python
tool_batch_ouput = web_search.batch(ai_msg.tool_calls)

print(tool_batch_output)
# [ToolMessage(content='[{"url": ...}]')]
```

---

## 🍝 Few-shot 프롬프팅

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 LLM 모델에게 몇가지 예시를 제공하여 원하는 작업 수행방식이나 출력형식을 보여주는 기법이다
</div>

🎯 모델에게 도구를 어떻게 사용해야 하는지 예시를 통해 보여줄 수도 있다.  
: (각 도구의 용도를 구분하여 few-shot 예제로 제시)

```
🧶 ToolCalling 적용 과정

1] 예시 생성
2] 프롬프트 템플릿 생성
3] 프롬프트 생성 및 모델 호출
4] 응답 파싱
```

```python
from langchain_core.messages import AIMessage, HumanMessage, ToolMessage
from langchain_core.prompts import ChatPromptTemplate
from langchain_google_genai import ChatGoogleGenerativeAI

examples = [
	HumanMessage("트러플 리조또의 가격과 특징, 그리고 어울리는 와인에 대해 알려주세요.", name="example_user"),
	AIMessage("메뉴 정보를 검색하고, 위키피디아에서 추가 정보를 찾은 후, 어울리는 와인을 검색해보겠습니다.", name="example_assistant"),
	AIMessage("", name="example_assistant", tool_calls=[{"name": "search_menu", "args": {"query": "트러플 리조또"}, "id": "1"}]),
	ToolMessage("트러플 리조또: 가격 ₩28,000, 이탈리아 카나롤리 쌀 사용, 블랙 트러플 향과 파르메산 치즈를 듬뿍 넣어 조리", tool_call_id="1"),
	AIMessage("트러플 리조또의 가격은 ₩28,000이며, 이탈리아 카나롤리 쌀을 사용하고 블랙 트러플 향과 파르메산 치즈를 듬뿍 넣어 조리합니다. 이제 추가 정보를 위키피디아에서 찾아보겠습니다.", name="example_assistant"),
	AIMessage("", name="example_assistant", tool_calls=[{"name": "wiki_summary", "args": {"query": "트러플 리조또", "k": 1}, "id": "2"}])
	ToolMessage("트러플 리조또는 이탈리아 요리의 대표적인 리조또 요리 중 하나로, 고급 식재료인 트러플을 사용하여 만든 크리미한 쌀 요리입니다. 주로 아르보리오나 카나롤리 등의 쌀을 사용하며, 트러플 오일이나 생 트러플을 넣어 조리합니다. 리조또 특유의 크리미한 질감과 트러플의 강렬하고 독특한 향이 조화를 이루는 것이 특징입니다.", tool_call_id="2"),
	AIMessage("트러플 리조또의 특징에 대해 알아보았습니다. 이제 어울리는 와인을 검색해보겠습니다.", name="example_assistant"),
	AIMessage("", name="example_assistant", tool_calls=[{"name": "search_wine", "args": {"query": "트러플 리조또에 어울리는 와인"}, "id": "3"}]),
	ToolMessage("트러플 리조또와 잘 어울리는 와인으로는 주로 중간 바디의 화이트 와인이 추천됩니다. 1. 샤르도네: 버터와 오크향이 트러플의 풍미를 보완합니다. 2. 피노 그리지오: 산뜻한 산미가 리조또의 크리미함과 균형을 이룹니다. 3. 베르나차: 이탈리아 토스카나 지방의 화이트 와인으로, 미네랄리티가 트러플과 잘 어울립니다.", tool_call_id="3"),
	AIMessage("트러플 리조또(₩28,000)는 이탈리아의 대표적인 리조또 요리 중 하나로, 이탈리아 카나롤리 쌀을 사용하고 블랙 트러플 향과 파르메산 치즈를 듬뿍 넣어 조리합니다. 주요 특징으로는 크리미한 질감과 트러플의 강렬하고 독특한 향이 조화를 이루는 점입니다. 고급 식재료인 트러플을 사용해 풍부한 맛과 향을 내며, 주로 아르보리오나 카나롤리 등의 쌀을 사용합니다. 트러플 리조또와 잘 어울리는 와인으로는 중간 바디의 화이트 와인이 추천됩니다. 특히 버터와 오크향이 트러플의 풍미를 보완하는 샤르도네, 산뜻한 산미로 리조또의 크리미함과 균형을 이루는 피노 그리지오, 그리고 미네랄리티가 트러플과 잘 어울리는 이탈리아 토스카나 지방의 베르나차 등이 좋은 선택이 될 수 있습니다.", name="example_assistant"),
]

system = """You are an AI assistant providing restaurant menu information and general food-related knowledge.
For information about the restaurant's menu, use the search_menu tool.
For other general information, use the wiki_summary tool.
For wine recommendations or pairing information, use the search_wine tool.
If additional web searches are needed or for the most up-to-date information, use the search_web tool.
"""

fewshot_prompt = ChatPromptTemplate.from_messages([
	("system", system),
	*examples,
	("human", "{query}"),
])

llm = ChatGoogleGenerativeAI(model="gemini-2.5-pro")

tools = [search_web, wiki_summary, search_wine, search_menu]
llm_with_tools = llm.bind_tools(tools=tools)

fewshot_search_chain = fewshot_prompt | llm_with_tools

query = "스테이크 메뉴가 있나요? 스테이크와 어울리는 와인을 추천해주세요."
response = fewshot_search_chain.invoke(query)

# 도구 사용 결과 출력
for tool_call in response.tool_calls:
	print(tool_call)
```
