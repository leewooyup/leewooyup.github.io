---
title: "[LangChain] Agent System 정의와 LangChain 프레임워크의 핵심 구성요소"
date: 2025-09-24 21:03 +0900
lastmod: 2024-09-24 21:03 +0900
categories: LangChain
tags:
  [ Agent, LangChain, LLM, Tool, runnable, message, prompt ]
---

## 🦙 Agent System

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 LLM을 추론엔진으로 사용하여 어떤 행동을 할지, 그 행동의 입력은 무엇일지 결정하는 시스템을 말한다
</div>

🎯 언어모델이 단순히 텍스트를 출력하는 것을 넘어 실제 행동을 취하게 한다.  
: 도구를 사용하는 과정까지 LLM이 컨트롤한다.

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🗿 Agent System 동작 방식</span>

```
행동의 결과를 다시 Agent에 피드백하여 추가 행동할지
또는, 작업을 완료할지를 LLM을 통해 판단해나가는 시스템이다.

즉, 사용자의 질문/지시사항에 대한 충분한 작업이 이루어졌다고 판단되면
최종적인 응답을 생성하는 프로세스이다.
```

대규모 언어모델 `(LLM)`을 활용하여 애플리케이션과 파이프라인을 신속하게 구축할 수 있게하는  
대표적인 프레임워크가 `🦜 LangChain` 이다.  

🦜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">LangChain</span> 의 설계철학  
: 에이전트도 하나의 도구(Tool) 처럼 취급 가능하다  
- 작은 단위: LLM Chain을 Tool로 감싸서 사용  
- 큰 단위: 여러 Tool을 다루는 Agent를 Tool로 감싸서 사용
: <span style="padding:3px 6px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🏆 조합성</span>이 올라간다.

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">⚔️ LangChain 프레임워크의 핵심 구성요소</span>

```
프롬프트
벡터 스토어
아웃풋 파서
도큐먼트 로더
도구
모델
메모리
텍스트 스프리터
```

`🦜 LangChain` 에서는 `Agent System`을 구현하기 위한 2가지 도구를 제공한다.  
- `create_tool_calling_agent` 함수  
    : 도구를 호출할 수 있는 agent를 생성하는 함수이다.

```python
from langchain.agents import create_tool_calling_agent

tools = [web_search_tool, blog_search, seach_menu]
agent = create_tool_calling_agent(llm, tools, agent_prompt)
```

- `AgentExecutor` 클래스  
    : Agent를 실행할 때 사용하는 클래스이다.

```python
from langchain.agents import AgentExecutor

agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
# verbose=True
# 중간의 agent 사고과정의 출력할 것인지를 설정

agent_response = agent_executor.invoke({"input": query})
```

```python
from langchain_core.prompts import ChatPromptTemplate, MessagePlaceholder
from langchain.agents import AgentExecutor, create_tool_calling_agent

agent_prompt = ChatPromptTemplate.from_messages([
    ("system", dedent("""
        You are an AI assistant providing restaurant menu information and general food-related knowledge. 
        Your main goal is to provide accurate information and effective recommendations to users.

        Key guidelines:
        1. For restaurant menu information, use the search_menu tool. This tool provides details on menu items, including prices, ingredients, and cooking methods.
        2. For general food information, history, and cultural background, utilize the wiki_summary tool.
        3. For wine recommendations or food and wine pairing information, use the search_wine tool.
        4. If additional web searches are needed or for the most up-to-date information, use the search_web tool.
        5. Provide clear and concise responses based on the search results.
        6. If a question is ambiguous or lacks necessary information, politely ask for clarification.
        7. Always maintain a helpful and professional tone.
        8. When providing menu information, describe in the order of price, main ingredients, and distinctive cooking methods.
        9. When making recommendations, briefly explain the reasons.
        10. Maintain a conversational, chatbot-like style in your final responses. Be friendly, engaging, and natural in your communication.


        Remember, understand the purpose of each tool accurately and use them in appropriate situations. 
        Combine the tools to provide the most comprehensive and accurate answers to user queries. 
        Always strive to provide the most current and accurate information.
    """)),
    # 대화기록 Agent에 반영
    MessagePlaceholder(variable_name="chat_history", optional=True),
    ("human", "{input}"),
    MessagePlaceholder(variable_name="agent_scratchpad"),
])
# agent_scratchpad
# agent의 사고과정과 중간 단계를 기록 (이전단계의 결과 + 다음 단계를 계획)

# 다음과 같은 도구가 있다고 가정
tools = [search_web, wiki_summary, search_wine, search_menu]
agent = create_tool_calling_agent(llm, tools, agent_prompt)

agent_executor = AgentExecutor(agent=agent, tools=tools, verbose=True)
```

`🦜 LangChain`에서 `Agent`는 <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 LLM이 도구를 선택/호출해서 최종 응답을 생성하는 실행 주체</span> 이고
`Agent`가 동적인 의사결정을 하는 시스템을 `Agent System` 이라고 부른다.  
이를 위해 `Agent`는 단순 LLM을 호출하는게 아닌, 여러 단계를 거치는 실행 루프로 구성되어 있다.  

1. 프롬프트 구성
2. LLM 호출 (어떤 Tool을 쓸지 결정)
3. Tool 실행
4. Tool 결과를 사용자 입력과 함께 다시 LLM 전달
5. 최종 답변 생성

크게 위와 같은 단계를 거치는데, 각 단계를 체인으로 구현할 수 있다.  
따라서, 체인을 구성하는 방식에 대해 알고 있어야 한다.

---

## 🪵 체인 구성을 위한 LCEL과 Runnable 객체

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 LCEL은 체인을 정의하고 실행하기 위한 표현언어이다
</div>

`|`로 연결해서 파이프라인을 만드는데 그 요소는 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Runnable</span> 객체</span>여야 한다.  
(한 구성요소의 출력을 다음 구성요소의 입력으로 전달한다)

```
🧶 Runnable 객체

랭체인에서 무언가를 실행하거나 연결할 수 있는 공통 인터페이스이다.
LCEL의 기본 단위로 LLM, 프롬프트 템플릿, 메시지, 체인, 툴 등 대부분이 Runnable로 구현되어 있다.

🎯 동일한 메서드로 실행가능 (ex. invoke...)
🎯 래핑 코드가 필요없음, 체인처럼 연결 가능
🎯 동기실행, 비동기실행, 스트리밍, 배치실행 같은 실행모드를 공통 인터페이스로 제공
    (invoke, stream, batch, ainvoke, astream, abatch)
🎯 Runnable 단위로 실행하면, 각 단계의 입/출력을 체계적으로 추적 가능
🎯 LangSmith 같은 툴과 쉽게 통합
```

`🍪 RunnableLambda`  
: 일반 파이썬 함수를 `Runnable` 형태로 감싸주는 어댑터이다.  
🎯 중간단계 전/후처리할 때 쓰인다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(대소문자 변환, 공백제거, 간단한 로직을 파이프라인에 끼워넣고 싶을 때)</span>

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnableLambda
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini 모델 초기화
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")
prompt = ChatPromptTemplate.from_template("Translate this to French: {text}")

strip_spaces = RunnableLambda(lambda x: x.strip())

chain = prompt | llm | strip_spaces

result = chain.invoke({"text": "hello world"})
print(result)
```

`🍪 RunnalbeParallel`  
: 병렬 실행

```python
from langchain_core.runnables import RunnableParallel
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini 모델 초기화
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

chain1 = (
    PromptTemplate.from_template("{country}의 수도는 어디야?")
    | llm
    | StrOutputParser()
)

chain2 = (
    PromptTemplate.from_template("{country}의 면적은 얼마야?")
    | llm
    | StrOutputParser()
)

combined = RunnableParallel(capital=chain1, area=chain2)
# 병렬처리
combined.invoke({"country": "대한민국"})

# 배치에서의 병렬 처리
combined.batch([{"country": "대한민국"}, {"country": "미국"}])
```

`🍪 RunnablePassthrough`  
: 입력을 그대로 출력하는 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">Runnable</span> 객체</span>이다.  
아무런 가공없이 그냥 흘려보내는 역할을 한다.  
🎯 입력을 그대로 전달해야 할 때 `Runnable`이 필요하다.  
🎯 병렬 실행에서 유용하다.

```python
from langchain_core.runnables import RunnablePassthrough, RunnableParallel
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini 모델 초기화
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

chain = RunnableParallel({
    "original": RunnablePassthrough(),
    "translation": llm
})

result = chain.invoke("How are you?")
print(result)
```

---

## 🦙 메시지 (BaseMessage)

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 LLM과 소통하는 기본단위로 메시지를 객체로 관리한다 (Runnable 객체)
</div>

채팅모델에 구조화된 메시지 객체 (역할, 메시지) 의 리스트를 입력으로 보내고  
AI응답을 나타내는 메시지를 반환 받는다.  
(한 턴의 질문 답변으로 끝나는 시스템이 아닌, 채팅처럼 AI와 메시지를 계속 주고받는 형태)  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">※ 문자열을 넘겨도 내부적으로 리스트로 변환된다.</span>  

BaseMessage를 상속받아 역할별로 메시지 객체가 구현되어 있다.  
<span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(BaseMessage의 필수값: content)</span>  

```
SystemMessage: 시스템의 역할 지정하는 메시지, 페르소나 지정에 사용
HumanMessage: 사용자의 입력을 나타내는 메시지
AIMessage: 채팅모델의 응답을 나타내는 메시지
ToolMessage: 도구 호출 결과를 나타내는 메시지
```

---

## 🦙 프롬프트 (prompt)

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 프롬프트를 객체로 관리한다. (Runnable 객체)
</div>

프롬프트를 객체로 관리하면 <span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🏆 재사용 및 유지보수</span>가 용이하고,  
동적으로 입력처리할 수 있다.

```
🧶 PromptTemplate

파이썬의 문자열 포맷팅을 사용하여 동적으로 특정 위치에 변수를 포함시켜
프롬프트를 구성할 수 있는 문자열 템플릿이다.

🎯 단순한 문자열 기반 변환, 요약, 단일 Q&A 프롬프트를 생성할 때 사용한다.

✏️ 템플릿 정의 방법
- from_template(): 문자열로 바로 생성
- PromptTemplate 생성자 호출
- load_prompt(): 파일에서 템플릿 불러오기
- partial_variables: 일부 변수를 고정, 하위-프롬프트 생성 가능
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🗿 프롬프트 구성</span>

```
- 지시: LLM에게 어떤 작업을 수행하도록 요청하는 구체적인 지시
- 예시: 요청된 작업을 수행하는 방법에 대한 예시
- 맥락: 특정 작업을 수행하기 위한 추가적인 맥락
- 질문: 어떤 답변을 요구하는 구체적인 질문
```

`🪄 partial_variables`  
: 항상 공통된 방식으로 가져오고 싶은 변수가 있는 경우 (ex. 날짜, 시간)  
사용자에게 입력받는게 아니라, 현재 날짜를 반환하는 함수를 사용하여 프롬프트를 부분적으로 변경할 때

```python
from datetime import datetime
from langchain_google_genai import ChatGoogleGenerativeAI

llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

def get_today():
    return datetime.now().strftime("%B %d")

prompt = PromptTemplate(
    template="오늘 날짜는 {today}입니다. 오늘이 생일인 유명인 {n}명을 나열해주세요. 생년월일을 표기해주세요.",
    input_variables=["n"],
    partial_variables={
        "today": get_today
    }
)

chain = prompt | llm
print(chain.invoke(3).content)
```

```
🧶 ChatPromptTemplate

대화목록을 프롬프트로 주입하고자 할 때
튜플형태 (role, message)의 메시지의 리스트를 넘긴다.

🎯 역할 기반 대화, 멀티턴, 시스템 지침 포함 시나리오 구현 시 유리하다.

LLM 모델이 역할에 대한 이해를 명확히 할 수 있다.
따라서, 역할 기반 응답이 더 자연스러울 수 있다.
ChatPromptTemplate, PromptTemplate이 할 수 있는게 각각 존재하여 사용하는 것이 아니라,
텍스트마다 역할을 지정하여 강조하고 싶을 때 ChatPromptTemplate를 사용한다.

단, PromptTemplate은 멀티턴 대화 구현이 어렵다.

🦸‍♂️ 역할 (role)
- "system": 시스템 설정 메시지, 전역 설정과 관련된 메시지
- "human": 사용자 입력 메시지
- "ai": AI 답변 메시지

```

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_google_genai import ChatGoogleGenerativeAI

# Gemini 모델 초기화
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

chat_template = ChatPromptTemplate.from_messages(
    [
        ("system", "당신은 친절한 AI 어시스턴트입니다. 당신의 이름은 {name} 입니다."),
        ("human", "반가워요!"),
        ("ai", "안녕하세요! 무엇을 도와드릴까요?"),
        ("human", "{user_input}"),
    ]
)

chat_prompt_format = chat_template.format_messages(name="영희")
print(chat_prompt_format)

chain = chat_template | llm
chain.invoke({"name": "Teddy", "user_input": "당신의 이름은 무엇입니까?"}).content
```