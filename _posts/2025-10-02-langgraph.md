---
title: "[LangGraph] 동일한 상태가 공유되는 랭그래프와 그래프 구성"
date: 2025-10-02 20:48 +0900
lastmod: 2025-10-02 20:48 +0900
categories: LangGraph
tags:
  [state, node, edge, TypedDict, BaseModel, Generic Type, Reducer, MessageGraph, feedback loop]
---

## 🪵 LangChain의 한계

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 복잡한 작업 플로우를 설계하기 힘들다
</div>

`Agent`가 동적인 의사결정 중에 각 단계에서 의도하지 않은 결과가 나왔을 때,  
이전 단계로 돌아가서 검색을 다시 한다거나, 답변을 다시 생성하는 플로우를 짜는게  
전통적인 RAG, `LangChain` 만으로는 한계가 있다.  

`LangGraph`를 이용하면 재생성 플로우를 설계해서 구현할 수 있다.  
즉, <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 복잡한 작업 흐름을 상태와 전이로 모델링</span> 해서 유연한 플로우를 짤 수 있다.  

## 🦙 그래프 구축, StateGraph

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">StateGraph</span>는 상태 기반 그래프를 정의하는 클래스이고, 상태를 중심으로 동작한다
</div>

```
🧶 상태 (State)

애플리케이션의 현재 스냅샷을 나타내는 공유 데이터 구조를 말한다.
각 노드에는 동일한 상태가 공유된다.
즉, 전체 워크플로의 컨텍스트를 관리하는 중앙 저장소 역할을 한다.

상태의 스키마 (schema)를 정의할 때,
typing.TypedDict나 pydantic.BaseModel로 정의할 수 있다.

```

보통 실무에서는 `TypedDict` 방식을 권장한다.

```
🎲 실무에서 TypedDict를 더 자주 쓰는 이유

LangGraph는 상태(state)를 단순히 key-value dict처럼 다루는 것을 전제로 만들어졌다.
내부적으로도 상태 병합, 직렬화 등을 할 때, dict 기반으로 처리한다.

TypedDict는 그냥 dict의 타입 힌트 버전이라 생각하면 된다.
때문에, LangGraph의 구조랑 자연스럽게 맞고, 불필요한 오버헤드 검증이 없어서 가볍다.

BaseModel은 입력 데이터 유효성 검증, 직렬화, 기본값 관리 같은 기능을 제공하지만 그만큼 성능 오버헤드가 생긴다.

LangGraph의 상태는 매 단계마다 계속 업데이트/머지되는데,
여기서 검증이 매번 일어나면 느려질 수 있다.

데이터 무결성 보장이나 외부 연동이 필요한 부분만 BaseModel을 섞어 쓰면 된다.
필드값에 제약 (ex. 범위 등)이 필요하면 BaseModel을 사용한다. (타입 안전성 보장)
```

```python
from typing import TypedDict

class GraphState(TypedDict):
    messages: list[str]
    score: float
``` 

```python
from pydantic import BaseModel, Field

class GraphState(BaseModel):
    messages: list[str] = Field(default_factory=list, description="대화 이력 메시지 리스트")
    score: float = Field(default=0.0, ge=0.0, le=1.0, description="점수 (0.0 ~ 1.0)")
```

```
🥔 파이썬 내장 타입 list와 typing.List 의 차이

Python 3.9 이상에서는 제네릭 지원 -> list[str] 사용 가능
3.8 이하에서는 제네릭 타입 표기가 불가능 했음, 그 당시 List[str]로 제네릭 타입 표기

따라서, 레거시 호환성목적에서만 typing.List 사용

🥔 제네릭 타입

안에 어떤 타입을 담을 수 있는지를 매개변수로 받는 타입
list는 어떤 타입이든 담을 수 있는 컨테이너 타입
list[str]은 문자열만 담는 리스트라는 구체적인 타입 명시
```

---

```
🧶 노드 (Node)

그래프의 기본 실행 단위로,
특정 작업을 수행하는 함수나 Agent를 나타낸다.

현재 상태를 입력으로 받고,
특정 작업을 수행해서
업데이트된 상태를 반환한다.
즉, 노드는 상태의 변환작업을 하는 함수이다.
```

```
🧶 엣지 (Edge)

현재 상태를 기반으로 다음에 실행할 노드를 결정하는 함수이다.
조건부 분기나 고정 전이를 통해 시스템 흐름을 제어한다.
```

---

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🌿 그래프 구성 기본 틀</span>  

```python
from typing import TypedDict

# 그래프에 사용될 공용 상태 schema 정의
# 1가지 필드를 가진 상태가 공유된다.
class OverallState(TypedDict):
    text: str

# 노드 (함수)
def node(state: OverallState):
    return {"text": "안녕하세요."} # 상태를 업데이트

# 그래프 구축
builder = StateGraph(OverallState)
builder.add_node(node)

builder.add_edge(START, "node") # node 함수로 시작
builder.add_edge("node", END) # node 함수 실행 후 그래프 종료

graph = builder.compile()

# 그래플 호출 및 초기 상태값 전달
graph.invoke({"text": ""})
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🗿 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">StateGraph</span> 내부 동작</span>  

```python
class StateGraph:
    def __init__(self, state_cls):
        # 상태 객체 생성, 클래스 호출 -> 인스턴스 생성
        self.state = state_cls()
        self.nodes = []
    
    def add_node(self, node_cls):
        # 노드 생성 시 상태 객체 주입
        node = node_cls(state=self.state)
        self.nodes.append(node)
    
    def run(self):
        # 모든 노드를 순서대로 실행
        for node in self.nodes:
            node.run()
```

## 🔌 LangGraph 예제: 레스토랑 메뉴 추천

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #336666;color:#4A4F5A;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🔌 상태를 정의하고, 각각의 역할을 맡은 함수 (노드)를 정의하고 이를 기반으로 그래프를 구성한다
</div>

```python
from typing import TypedDict, List, Literal
from langchain_chroma import Chroma
from langchain_ollama import OllamaEmbeddings
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_google_genai import ChatGoogleGenerativeAI
from langgraph.graph import StateGraph, START, END

# LLM 모델
llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

embeddings_model = OllamaEmbeddings(model="bge-m3")

# Chroma 인덱스 로드
vector_db = Chroma(
    embedding_function=embeddings_model,
    collection_name="restaurant_menu",
    persist_directory="./chroma_db"
)

class MenuState(TypedDict):
    user_query: str
    is_menu_related: bool
    search_results: List[str]
    final_answer: str

# 사용자로부터 입력을 받는 노드
def get_user_query(state: MenuState) -> MenuState:
    user_query = input("무엇을 도와드릴까요?")
    return {"user_query": user_query}

# 사용자 의도(intent) 분류 노드
def analyze_input(state: MenuState) -> MenuState:
    analyze_template = """
    사용자의 입력을 분석하여 레스토랑 메뉴 추천이나 음식 정보에 관한 질문인지 판단하세요.

    사용자 입력: {user_query}

    레스토랑 메뉴나 음식정보에 관한 질문이면 "True", 아니면 "False"로 답변하세요.

    답변:
    """

    analyze_prompt = ChatPromptTemplate.from_template(analyze_template)
    analyze_chain = analyze_prompt | llm | StrOutputParser()

    result = analyze_chain.invoke({"user_query": state['user_query']})
    is_menu_related = result.strip().lower() == "true"

    return {"is_menu_related": is_menu_related}

# 벡터 db 검색 노드 (retriever)
def search_menu_info(state: MenuState) -> MenuState:
    results = vector_db.similarity_search(state['user_query'], k=2)
    search_results = [doc.page_content for doc in results]
    
    return {"search_results": search_results}

# 메뉴 관련 응답 생성 노드 (response generator)
def generate_menu_response(state: MenuState) -> MenuState:
    response_template = """
    사용자 입력: {user_query}
    메뉴 관련 검색 결과: {search_results}

    위 정보를 바탕으로 사용자의 메뉴 관련 질문에 대한 상세한 답변을 생성하세요.
    검색 결과의 정보를 활용하여 정확하고 유용한 정보를 제공하세요.

    답변:
    """
    response_prompt = ChatPromptTemplate.from_template(response_template)
    response_chain = response_prompt | llm | StrOutputParser()

    final_answer = response_chain.invoke(
        {
            "user_query": state['user_query'],
            "search_results": state['search_results']
        }
    )
    print(f"\n메뉴 어시스턴트: {final_answer}")

    return {"final_answer": final_answer}

# 일반 응답 생성 노드 (reponse generator)
def generate_general_response(state: MenuState) -> MenuState:
    response_template = """
    사용자 입력: {user_query}

    위 입력은 레스토랑 메뉴나 음식과 관련이 없습니다.
    일반적인 대화 맥락에서 적절한 답변을 생성하세요.

    답변:
    """
    response_prompt = ChatPromptTemplate.from_template(response_template)
    response_chain = response_prompt | llm | StrOutputParser()

    final_answer = response_chain.invoke({"user_query": state['user_query']})

    print(f"\n일반 어시스턴트: {final_answer}")
    
    return {"final_answer": final_answer}

# 라우터 노드 (조건부 엣지 함수): 다음 노드 결정, 검색 or 일반 답변
def decide_next_step(state: MenuState) -> Literal["search_menu_info", "generate_general_response"]:
    if state['is_menu_related']:
        return "search_menu_info"
    else:
        return "generate_general_response"

# 그래프 구성
builder = StateGraph(MenuState)
builder.add_node("get_user_query", get_user_query)
builder.add_node("analyze_input", analyze_input)
builder.add_node("search_menu_info", search_menu_info)
builder.add_node("generate_menu_response", generate_menu_response)
builder.add_node("generate_general_response", generate_general_response)

builder.add_edge(START, "get_user_query")
builder.add_edge("get_user_query", "analyze_input")
builder.add_conditional_edges(
    "analyze_input",
    decide_next_step,
    {
        "search_menu_info": "search_menu_info",
        "generate_general_response": "generate_general_response"
    }
)
builder.add_edge("search_menu_info", "generate_menu_response")
builder.add_edge("generate_menu_response", END)
builder.add_edge("generate_general_response", END)

graph = builder.compile()

while True:
    initial_state = {'user_query': ""}
    graph.invoke(intial_state)
    continue_chat = input("다른 질문이 있으신가요? (y/n): ").lower()
    if continue_chat != 'y':
        print("대화를 종료합니다. 감사합니다..")
        break
```

## 🦙 State Reducer

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">State Reducer</span>는 상태 업데이트를 관리하는 함수이다
</div>

기본으로 설정되어 있는 리듀서에서는 각 노드의 반환값을 해당 상태 key의 이전값을 덮어쓰기 한다.  
`Add Reducer`는 메시지 리스트에서 이전 상태에 새로운 값을 추가할 때 사용한다.  

```python
from typing import TypedDict, List, Annotated

class ReducerState(TypedDict):
    query: str
    documents: Annotated[List[str], add]
    # Annotated를 사용해서 add 리듀서를 추가한다.
```

이 밖에도 특수한 상태 관리를 위해, 사용자 정의 리듀서를 지원한다. <span style="padding:0 3px;font-size:16px;border-radius:5px;background-color:#F7F7F7;color:#34343c;">(custom 리듀서)</span>  
ex) 중복제거, 정렬, 최대/최소값 유지, 조건부 병합 등

```python
from typing import TypedDict, List, Annotated

def reduce_unique_documents(left: list | None, right: list | None) -> list:
    """
    Combine two lists of documents, removing duplicates.
    """
    if not left:
        left = []
    if not right:
        right = []
    return list(set(left + right))

class CustomReducerState(TypedDict):
    query: str
    documents: Annotated[List[str], reduce_unique_documents]

def node_1(state: CustomReducerState) -> CustomReducerState:
    print("---Node 1 (query update)---")
    query = state["query"]
    return {"query": query}

def node_2(state: CustomReducerState) -> CustomReducerState:
    print("---Node 2 (add documents)---")
    return {"documents": ["doc1.pdf", "doc2.pdf", "doc3.pdf"]}

def node_3(state: CustomReducerState) -> CustomReducerState:
    print("---Node 3 (add more documents)---")
    return {"documents": ["doc2.pdf", "doc4.pdf", "doc5.pdf"]}

builder = StateGraph(CustomReducerState)
builder.add_node("node_1", node_1)
builder.add_node("node_2", node_2)
builder.add_node("node_3", node_3)

builder.add_edge(START, "node_1")
builder.add_edge("node_1", "node_2")
builder.add_edge("node_2", "node_3")
builder.add_edge("node_3", END)

graph = builder.compile()

initial_state = {"query": "채식주의자를 위한 비건음식을 추천해주세요.", "documents": []}

final_state = graph.invoke(initial_state)
print("최종 상태:", final_state)

```

---

```
🧶 MessageGraph

LangChain의 ChatModel을 위한 특수한 형태의 StateGraph
즉, 메시지 리스트를 지원하는 StateGraph이다.
Message 객체 리스트 (HumanMessage, AIMessage 등)를 입력으로 처리한다.

🎯 대화 기록(메시지 리스트)을 효과적으로 관리해서 활용할 수 있다.
- 자연스러운 대화 흐름
- 컨텍스트 활용 답변

[방법1]
Message State 정의
1] 대화기록을 그래프 상태에서 메시지 목록으로 저장
2] Message 객체 목록을 저장하는 messages 키 추가
3] 해당 키에 리듀서 함수를 추가하여 메시지 업데이트를 관리

[방법2]
LangGraph의 MessageState라는 미리 만들어진 상태를 이용
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">[방법1] <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">graph_state.py</span></span>

```python
from typing import Annotated
from langchain_core.messages import AnyMessage
from langgraph.graph.message import add_messages

class GraphState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    # add [operator]: 새 메시지를 기존 목록에 단순히 추가
    # add_messages [function]
    #   - 새 메시지는 기존 목록에 추가
    #   - 기존 메시지 업데이트도 처리 (메시지 ID를 추적)
```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">[방법2] <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">graph_state.py</span></span>

```python
from langgraph.graph import MessagesState
from typing import List
from langchain_core.documents import Document

class GraphState(MessagesState):
    # messages 키는 기본 제공

```

---

## 🍝 답변 품질평가 및 피드백 루프

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #9B111E;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🍝 조건부 엣지를 활용해서 평가결과를 바탕으로 응답을 재생성하는 과정을 반복할 수 있다
</div>

🍍 `Iteration/Feedback Loop`  
: 조건부 엣지를 활용해서 반복적인 작업을 수행하는 것을 말한다.  

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">🌿 RAG Chain</span> 구성</span>  

```python
from langchain_chroma import Chroma
from langchain_ollama import OllamaEmbeddings
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.messages import HumanMessage, AIMessage
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough, RunnableLambda

embeddings_model = OllamaEmbeddings(model="bge-m3")

vector_db = Chroma(
    embedding_function=embeddings_model,
    collection_name="restaurant_menu",
    persist_directory="./chroma_db"
)

llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

system = """
You are a helpful assistant. Use the following context to answer the user's question:

[Context]
{context}
"""

prompt = ChatPromptTemplate.from_messages([
    ("system", system),
    ("human", "{question}")
])

retriever = vector_db.as_retriever(
    search_kwargs={"k": 2}
)

def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | prompt
    | llm
    | StrOutputParser()
)

query = "채식주의자를 위한 메뉴를 추천해주세요."
response = rag_chain.invoke(query)

print(response)

```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;"><span style="font-family:var(--bs-font-monospace);font-size:.85rem;">♻️ LangGraph,</span> 답변 품질 평가 함수를 통한 피드백 루프 구성</span>  

```python
from typing import Annotated, List, Literal
from langchain_core.messages import AnyMessage
from langchain_core.documents import Document
from langgraph.graph.message import add_messages

class GraphState(TypedDict):
    messages: Annotated[list[AnyMessage], add_messages]
    documents: List[Document]
    grade: float
    num_generation: int # 답변 생성횟수 카운트

class GradeResponse(BaseModel):
    """A score for answers"""
    score: float = Field(..., ge=0, le=1, description="A score from 0 to 1, wher 1 is perfect")
    explanation: str = Field(..., description="An explanation for the given score")

# RAG 수행 함수 (노드)
def retrieve_and_respond(state: GraphState):
    last_human_message = state['message'][-1]

    query = last_human_message.content

    # 문서 검색
    retrieved_docs = retriever.invoke(query)

    # 응답 생성
    response = rag_chain.invoke(query)

    return {
        "messages": [AIMessage(content=response)], # add messages 리듀서 적용
        "documents": retrieved_docs # 기본 리듀서 적용, 최종 상태 업데이트 (가장 마지막에 검색된 문서)
    }

# 답변 품질 평가 함수 (노드)
def grade_answer(state: GraphState):
    messages = state['messages']
    question = message[-2].content
    answer = message[-1].content
    context = format_docs(state['documents'])

    grading_system = """
    You are an expert grader.
    Grade the following answer based on its relevance and accuracy to the question, considering the given context.
    Provide a score from 0 to 1, where 1 is perfect, along with an explanation
    """

    grading_prompt = ChatPromptTemplate.from_messages([
        ("system", grading_system),
        ("human", "[Question]\n{question}\n\n[Context]\n{context}\n\n[Answer]\n{answer}\n\n[Grade]\n")
    ])

    grading_chain = grading_prompt | llm.with_structured_output(schema=GradeResponse)

    grade_response = grading_chain.invoke({
        "question": question,
        "context": context,
        "answer": answer
    })

    num_generation = state.get('num_generation', 0)
    num_generation += 1

    return {"grade": grade_response.score, "num_generation": num_generation}

# 조건부 엣지 함수
def should_retry(state: GraphState) -> Literal["retrieve_and_respond", "generate"]:
    print("---GRADING---")
    print("Grade Score:", state["grade"])

    # 답변 생성 횟수가 3회 이상이면 "generate"를 반환
    if state["num_generation"] > 2:
        return "generate"

    # 답변 품질 평가 점수가 0.7 미만이면 RAG 체인을 다시 실행
    if state["grade"] < 0.7:
        return "retrieve_and_respond"
    else:
        return "generate"

# 그래프 구성
builder = StateGraph(GraphState)
builder.add_node("retrieve_and_respond", retrieve_and_respond)
builder.add_node("grade_answer", grade_answer)

builder.add_edge(START, "retrieve_and_respond")
builder.add_edge("retrieve_and_respond", "grade_answer")
builder.add_conditional_edges(
    "grade_answer",
    should_retry,
    {
        "retrieve_and_respond": "retrieve_and_respond",
        "generate": END
    }
)

graph = builder.compile()

# graph 실행
# LangChain의 Message 객체를 원소로 갖는 리스트
initial_state = {
    "messages": [HumanMessage(content="채식주의자를 위한 메뉴를 추천해주세요.")]
}

final_state = graph.invoke(initial_state)
print("최종 상태:", final_state)
```