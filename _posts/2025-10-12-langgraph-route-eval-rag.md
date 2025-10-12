---
title: "[LangGraph] 검색 도구 라우팅 및 자기평가를 이용한 RAG 전략"
date: 2025-10-12 15:40 +0900
lastmod: 2025-10-12 15:40 +0900
categories: LangGraph
tags:
  [
    RAG,
    routing,
    grader,
    rewriter,
    hallucination,
    prompt,
    feedback loop,
    fallback
  ]
---

## 🪵 기본 RAG 구조 개선

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid #FFD24D;padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🪵 어떤 단계에 초점을 맞춰 개선하느냐에 따라 다양한 RAG 전략이 있다
</div>

`Adaptive RAG`는 질문에 따라 검색 전략 (강도, 범위)을 적응적으로 바꾸는데 주로 초점을 맞추었다면,  
`Self RAG`는 검색 및 답변 과정 (검색 여부, 검색/답변 횟수, 내용)을 LLM 모델이 판단하고 통제하는데 초점이 맞춰져 있다.  

| 검색 전략 | 목적 |
| ----- | ------- |
| Adaptive RAG | 검색 전략 최적화 |
| Self RAG | 자기평가 기반 검색/답변 통제 |

굳이 나눠서 생각하기보다는, <span style="font-size:0.98rem;padding:3px 5px;background-color:#ffdce0;border-radius:4px;color:#34343c;font-weight:normal;">🍞 어느 단계에 초점을 맞춰 최적화했는지</span> 를 고려하면  
각 RAG 전략의 목적을 알기 쉬울 것 같다.  


## 🦙 Adaptive RAG

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 <span style="font-family:var(--bs-font-monospace);font-size:.98rem;">Adaptive RAG</span> 는 질문 복잡성에 따라 필요한 검색을 동적으로 선택하는 방법을 말한다.
</div>

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">⛓️ LLM 체인 정의</span>  

```python
# 라우팅 결정을 위한 데이터 모델 정의
class ToolSelector(BaseModel):
    """
    Routes the user question to the most appropriate tool.
    """
    tool: Literal["search_menu", "search_wine", "search_web"] = Field(
        description="Select one of the tools: search_menu, search_wine or search_web based on the user's question."
    )

# 구조화된 출력 형태를 위한 LLM 설정
structured_llm = llm.with_structured_output(ToolSelector)

# 라우팅을 위한 시스템 프롬프트 정의
system_prompt = dedent("""
You are an AI assistant specializing in routing user questions to the appropriate tool.
Use the following guidelines:
- For questions about the restaurant's menu, use the search_menu tool.
- For wine recommedations or pairing information, use the search_wine tool.
- For any other information or the most up-to-date data, use the search_web tool.
Always choose the most appropriate tool based on the user's question.
""")

route_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "{question}")
    ]
)

question_router_chain = route_prompt | structured_llm

# 검색된 문서가 있었을 경우
# 답변 생성을 위한 시스템 프롬프트 정의
system_prompt = """
You are an assistant answering questions based on provided documents. Follow these guidelines:
	
1. Use only information from the given documents.
2. If the document lacks relevant info, say "The provided documents don't contain information to answer this question."
3. Cite relevant parts of the document in your answer.
4. Don't speculate or add information not in the documents.
5. Keep answers concise and clear.
6. Omit irrelevant information.
"""

generate_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", "Answer the following question using these documents:\n\n[Documents]\n{documents}\n\n[Question]\n{question}")
    ]
)

generate_response_chain = generate_prompt | llm | StrOutputParser()

# 도구 실행이 없었거나, 실패했을 경우
# 일반적인 질문에 답변하도록 fallback_prompt 정의
system_prompt = """
You are an AI assistant helping with various topics. Follow these guidelines:
	
1. Provide accurate and helpful information to the best of your ability.
2. Express uncertainty when unsure; avoid speculation.
3. Keep answers concise yet informative.
4. Inform users they can ask for clarification if needed.
5. Respond ethically and constructively.
6. Mention reliable general sources when applicable.
"""

fallback_prompt = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human": "{question}")
    ]
)

fallback_chain = fallback_prompt | llm | StrOutputParser()

```

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🦜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">LangGraph</span>로 그래프 구성</span>  

```python
from typing import TypedDict, List
from langchain_core.documents import Document

# 상태 스키마 정의
class AdaptiveRagState(TypedDict):
    question: str
    documents: List[Document]
    generation: str

# LangGraph 그래프 초기화
builder = StateGraph(AdaptiveRagState)
    
# 노드 정의
# 질문분석 -> 라우팅
def route_question_adaptive(state: AdaptiveRagState) -> Literal["search_menu", "search_wine", "search_web", "llm_fallback"]:
    """ 질문 라우팅 함수 """

    question = state["question"]
    try:
        result = question_router_chain.invoke({"question": question})
        datasource = result.tool

        if datasource == "search_menu":
            return "serach_menu"
        elif datasource == "search_wine":
            return "search_wine"
        elif datasource == "search_web":
            return "search_web"
        else:
            return "llm_fallback"

    except Exception as e:
        print(f"Error in routing: {str(e)}")
        return "llm_fallback"

def search_menu(state: AdaptiveRagState):
    """
    레스토랑 메뉴 검색 함수
    """

    question = state["question"]
    docs = search_menu.invoke(question)
    if len(docs) > 0:
        return {"documents": docs}
    else:
        return {"documents": [Document(page_content="관련 메뉴 정보를 찾을 수 없습니다.")]}

def search_wine(state: AdaptiveRagState):
    """
    레스토랑 와인 검색 함수
    """

    # 생략... (거의 동일한 로직)

def search_web(state: AdaptiveRagState):
    """
    웹 검색 함수
    """

    # 생략... (거의 동일한 로직)

def generate_response(state: AdaptiveRagState):
    """
    검색된 문서를 기반으로 답변 생성 함수
    """

    question = state.get("question", None)
    documents = state.get("documents", [])
    if not isinstance(documents, list):
        documents = [documents]

    documents_text = "\n\n".join([f"--\n본문:{doc.page_content}\n메타데이터:{str(doc.metadata)}\n--" for doc in documents])

    # 문서기반 답변 생성 체인 호출...
    return {"generation": generation}

def llm_fallback_adaptive(state: AdaptiveRagState):
    """
    도구 호출에 실패하거나, 도구를 선택하지 않은 경우
    일반적인 질문 답변 생성 함수
    """

    question = state.get("question", "")

    # 일반 답변 생성 체인 호출...
    return {"generation": generation}

# 노드 추가
builder.add_node("search_menu", search_menu)
builder.add_node("search_wine", search_wine)
builder.add_node("search_web", search_web)
builder.add_node("generate_response", generate_response)
builder.add_node("llm_fallback", llm_fallback_adaptive)

# 그래프 구성
builder.add_conditional_edges(
    START,
    route_question_adaptive
)
builder.add_edge("search_menu", "generate")
builder.add_edge("search_wine", "generate")
builder.add_edge("search_web", "generate")
builder.add_edge("generate", END)

builder.add_edge("llm_fallback", END)

adaptive_rag = builder.compile()

display(Image(adaptive_rag.get_graph().draw_mermaid_png()))
```

---

## 🦙 Self RAG

<div style="margin-bottom:15px;font-size:20px;border-bottom: 2px solid rgb(35,43,47);padding-right:10px;overflow-x:auto;display:inline-block;white-space:nowrap;">
	🦙 검색과 답변에 대한 자체 평가 로직을 추가하여 질문과 관련성 높은 최종 답변 생성을 목표로 한다
</div>
 
1. 검색된 문서 관련성 평가 `[Retrieval Grader]` (검색된 청크가 질문에 유용한 정보를 제공하는지 판단, 관련 있는 청크만 필터링)
- 출력: `relevant`, `irrelevant`  
2. 생성된 답변의 환각 여부 평가 `[Hallucination Grader]` (생성된 응답이 청크의 정보를 근거하고 있는지 판단)  
- 출력: `supported`, `no_support`  
3. 생성된 답변의 유용성 평가 `[Answer Grader]` (응답 품질과 관련성을 수치화)  
- 출력: 1 (전혀 유용하지 않음) ~ 5 (매우 유용함) / `useful`, `not_useful`  

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">⛓️ LLM 체인 정의</span>  

```python
# 검색된 문서의 관련성 평가 결과를 위한 데이터 모델 정의
class BinaryGradeDocuments(BaseModel):
    """
    Binary score for relevance check on retrieved documents.
    """
    binary_score: str = Field(
        description="Documents are relevant to the question, 'yes' or 'no'"
    )

# 🦜 체인을 정의하는 함수를 만들고 그 안에 아래 코드를 작성

# [Retrieval Grader]
# 문서 관련성 평가를 위한 시스템 프롬프트 정의
system_prompt = """
You are an expert in evaluating the relevance of search results to user queries.

[Evaluation criteria]
1. 키워드 관련성: 문서가 질문의 주요 단어나 유사어를 포함하는지 확인
2. 의미적 관련성: 문서의 전반적인 주제가 질문의 의도와 일치하는지 평가
3. 부분 관련성: 질문의 일부를 다루거나 맥락 정보를 제공하는 문서도 고려
4. 답변 가능성: 직접적인 답이 아니더라도 답변 형성에 도움될 정보 포함 여부 평가

[Scoring]
- Rate 'yes' if relevant, 'no' if not
- Default to 'no' when uncertain

[Key points]
- Consider the full context of the query, not just word matching
- Rate as relevant if useful information is present, even if not a complete answer

Your evaluation is crucial for improving information retrieval systems. Provide balanced assessments.

# Answer 
"""

human_prompt = f"[Retrieved document]\n{document}\n\n[User question]\n{question}"

# [Hallucination Grader]
# 답변의 환각 평가를 위한 시스템 프롬프트 정의
system_prompt = """
You are an expert evaluator assessing whether an LLM-generated answer is grounded in and supported by a given set of facts.

[Your task]
    - Review the LLM-generated answer.
    - Determine if the answer is fully supported by the given facts.

[Evaluation criteria]
    - 답변에 주어진 사실이나 명확히 추론할 수 있는 정보 외의 내용이 없어야 합니다.
    - 답변의 모든 핵심 내용이 주어진 사실에서 비롯되어야 합니다.
    - 사실적 정확성에 집중하고, 글쓰기 스타일이나 완전성은 평가하지 않습니다.

[Scoring]
    - 'yes': The answer is factually grounded and fully supported.
    - 'no': The answer includes information or claims not based on the given facts.

Your evaluation is crucial in ensuring the reliability and factual accuracy of AI-generated responses. Be thorough and critical in your assessment.
"""

human_prompt = f"[Set of facts]\n{documents}\n\n[LLM generation]\n{generation}"

# [Answer Grader]
# 답변의 유용성 평가를 위한 시스템 프롬프트 정의
system_prompt = """
You are an expert evaluator tasked with assessing whether an LLM-generated answer effectively addresses and resolves a user's question.

[Your task]
    - Carefully analyze the user's question to understand its core intent and requirements.
    - Determine if the LLM-generated answer sufficiently resolves the question.

[Evaluation criteria]
    - 관련성: 답변이 질문과 직접적으로 관련되어야 합니다.
    - 완전성: 질문의 모든 측면이 다뤄져야 합니다.
    - 정확성: 제공된 정보가 정확하고 최신이어야 합니다.
    - 명확성: 답변이 명확하고 이해하기 쉬워야 합니다.
    - 구체성: 질문의 요구 사항에 맞는 상세한 답변이어야 합니다.

[Scoring]
    - 'yes': The answer effectively resolves the question.
    - 'no': The answer fails to sufficiently resolve the question or lacks crucial elements.

Your evaluation plays a critical role in ensuring the quality and effectiveness of AI-generated responses. Strive for balanced and thoughtful assessments.
"""

human_prompt = f"[User question]\n{question}\n\n[LLM generation]\n{generation}"

# [Question Re-writer]
# 검색에 최적화된 형태로 질의 재작성을 위한 시스템 프롬프트 정의
system_prompt = """
You are an expert question re-writer. Your task is to convert input questions into optimized versions 
for vectorstore retrieval. Analyze the input carefully and focus on capturing the underlying semantic 
intent and meaning. Your goal is to create a question that will lead to more effective and relevant 
document retrieval.

[Guidelines]
    1. 질문에서 핵심 개념과 주요 대상을 식별하고 강조합니다.
    2. 약어나 모호한 용어를 풀어서 사용합니다.
    3. 관련 문서에 등장할 수 있는 동의어나 연관된 용어를 포함합니다.
    4. 질문의 원래 의도와 범위를 유지합니다.
    5. 복잡한 질문은 간단하고 집중된 하위 질문으로 나눕니다.

Remember, the goal is to improve retrieval effectiveness, not to change the fundamental meaning of the question.
"""

human_prompt = f"[Initial question]\n{question}\n\n[Improved question]\n"

# 아래와 같은 코드로 각 단계 체인 정의
from langchain_google_genai import ChatGoogleGenerativeAI

llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash", temperature=0)
# LLM에 구조화된 출력 형태(객체 형태) 정의
structured_llm_grader = llm.with_structured_output(BinaryGraderAnswer)

prompt_template = ChatPromptTemplate.from_messages(
    [
        ("system", system_prompt),
        ("human", human_prompt)
    ]
)

grader_binary_chain = prompt_template | structured_llm_grader
rewriter_chain = prompt_template | llm | StrOutputParser()
```

생성된 응답이 검색된 문서 정보를 근거하고 있지만, 응답이 질문과의 연관성이 떨어진다면  
질문을 재작성해야 해서 `Question Re-writer` 체인이 필요하다.  

<span style="padding:3px 6px;font-size:17px;border-radius:5px;background-color:rgba(0,0,0,0.03);color:#3f596f;">🦜 <span style="font-family:var(--bs-font-monospace);font-size:.85rem;">LangGraph</span>로 그래프 구성</span>  

```python
# 상태 스키마 정의
class SelfRagState(TypedDict):
    question: str # 사용자 질문
    generation: str # LLM 생성 답변
    documents: List[Document] # 컨텍스트 문서(검색된 문서)
    num_generation: int # 질문 or 답변 생성 횟수 (무한 루프 방지에 활용)

# ***************
# ** 노드 정의 **
# ***************
def retrieve_menu(state: SelfRagState):
    """ 문서 검색 함수 """

    # 문서 검색 로직...
    return {"documents": documents}

def generate_response(state: SelfRagState):
    """ 답변을 생성하는 함수 """

    # 답변 생성 체인 호출...
    # 생성 횟수 업데이트
    num_generations = state.get("num_generations", 0)
    num_generations += 1
    return {
        "generation": generation,
        "num_generations": num_generations
    }

def grade_documents_self(state: SelfRagState):
    """ 검색된 문서의 관련성을 평가하는 함수 """

    filtered_docs = []
    for d in documents:
        # 문서 관련성 평가 체인 호출...
        if grade == 'yes':
            filtered_docs.append(d)

    return {"documents": filtered_docs} # 문서 관련성 평가에 합격한 문서들만 저장

def transform_query_self(state: SelfRagState):
    """ 질문을 개선하는 함수 """

    # 질문 재작성 체인 호출
    # 생성 횟수 업데이트
    num_generations = state.get("num_generations", 0)
    num_generations += 1
    return {
        "question": rewritten_question,
        "num_generations": num_generations
    }

# ***************
# ** 엣지 정의 **
# ***************
def decide_to_generate_self(state: SelfRagState):
    """ 답변 생성 여부를 결정하는 함수 (문서 관련성 평가 결과) """

    num_generations = state.get("num_generations", 0)
    if num_generations > 2:
        print("결정: 검색 및 질문 개선 횟수 초과, 답변 생성")
        return "generate"

    filtered_documents = state.get("documents", None)
    if not filtered_documents:
        print("결정: 모든 문서가 질문과 관련 없음, 질문 개선 필요")
        return "transform_query"
    else:
        print("결정: 답변 생성")
        return "generate"

def grade_generation_self(state: SelfRagState):
    """ 생성된 답변의 품질을 평가하는 함수 (답변의 환각여부 및 유용성 판단 결과) """

    num_generations = state.get("num_generations", 0)
    if num_generations > 2:
        print("결정: 답변 생성 횟수 초과, 종료")
        return "end"
    
    question, documents, generation = state["question"], state["documents"], state["generation"]

    # 1단계: 환각 여부 확인
    # 환각 평가 체인 호출...
    if hallucination_grade.binary_score == 'yes':
        print("결정: 환각이 없음, 생성된 답변이 검색된 문서를 근거로 함.")
        # 2단계: 답변 유용성 평가
        # 답변 유용성 평가 체인 호출
        if relevance_grade.binary_score == 'yes':
            print("결정: 생성된 답변이 질문에 유용한 정보임.")
            return "useful"
        else:
            print("결정: 생성된 답변이 질문에 유용하지 않음.")
            return "not useful"
    else:
        print("결정: 환각이 있음, 생성된 답변이 검색된 문서에 근거하지 않음.")
        return "not supported"

# 상태기반 그래프 초기화
builder = StateGraph(SelfRagState)

# 노드 추가
builder.add_node("search_menu", retrieve_menu)
builder.add_node("grade_documents", grade_documents_self) # 문서 관련성 평가
builder.add_node("generate_response", generate_response)
builder.add_node("transform_query", transform_query_self) # 질문 쿼리 개선

# 그래프 구축
builder.add_edge(START, "search_menu")
# 문서 평가
builder.add_edge("search_menu", "grade_documents")

# 문서 평가 후 결정
builder.add_conditional_edges(
    "grade_documents",
    decide_to_generate_self,
    {
        "transform_query", "transform_query",
        "generate": "generate_response",
    },
)
# feedback loop 형성
builder.add_edge("transform_query", "search_menu")

# 답변 생성 후 평가
builder.add_conditional_edges(
    "generate_response",
    grade_generation_self,
    {
        "not_supported": "generate_response", # 환각 발생 => 답변 재생성
        "not_useful": "transform_query", # 답변의 유용성 부족 => 질문 쿼리 개선, 재검색
        "useful": END,
        "end": END,
    }
)

# 컴파일
self_rag = builder.compile()

# 그래프 시각화
display(Image(self_rag.get_graph().draw_mermaid_png()))
```