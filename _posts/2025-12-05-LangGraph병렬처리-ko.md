---
title: "[인턴] LangGraph에서 병렬 처리 실행하기"
date: 2025-12-05 14:50:00 +09:00
last_modified_at: 2025-12-05 14:50:00 +09:00
categories: [인턴]
tags: [인턴], [LangGraph], [Internship]
description: LangGraph에서 특정 노드를 병렬 처리하고 싶을 때
language: ko
postid: 11
---
# 문제 상황
- 현재 사용자가 업로드한 문서 종류 확인 - 스캔 - 문서 내에서 필요한 값 추출 - 시스템 값과 비교해 결과 반환의 로직을 구축하고 있음
- 사용자가 업로드한 문서에 여러 개의 첨부 파일이 있는 경우에 대응하기 위해 병렬 처리가 필요해짐

# 핵심
- LangGraph에서 Send를 사용하면 병렬 처리를 알아서 해준다.
- 병렬 처리된 결과가 여러 개가 될 수 있는 state의 값은 모두 Annotated와 operator.add를 사용해 처리해야 오류가 나지 않는다.

# 참고

[Understaning Send in LangGraph](https://medium.com/@syeedmdtalha/understanding-send-in-langgraph-573f4d7c9a0c)
```python
# State 관리
from operator import add
from typing import Annotated, TypedDict

# LangGraph
from langgraph.graph import StateGraph, START, END
from langgraph.types import Send  # 병렬 처리


# State 선언
class GraphState(TypedDict):
    attached_files: list
    is_file_attaced: bool
    processing_file: str | bytes
    ocr_result: Annotated[list[str], add]
    extract_result: Annotated[list[float], add]

def file_processor():
    ### file_processor

def ocr():
    is_file_attached = state.get("is_file_attached", False)
    if is_file_attached:
      ### OCR Processing
    return {"ocr_result": [ocr_result]}

def extractor():
    ### extractor Processing
    return {"extractor_result": [extractor]}

def splitter(state:GraphState):  # 2
    sends = []
    attached_files = state['attached_files']
    for file in attached_files:
        sends.append(Send("ocr", {"processing_file": file, "is_file_attached": True))  # 1
    return sends
    
graph = StateGraph(GraphState)

graph.add_node("file_processor", file_processor)
graph.add_node("ocr", ocr)
graph.add_node("extractor", extractor)

graph.add_edge(Start, "file_processor")
graph.add_conditional_edges("file_processor", splitter, ["ocr"])  # 3
graph.add_edge("ocr", "extractor")

graph = builder.compile()

```
- `Send` 사용 시 전체 state가 아닌 병렬 처리할 state만 보내도 됨 (# 1)
  - 넘어간 요소들만 노드에서 사용 가능
    - 병렬 처리 할 state(bytes로 된 file 객체들) 외에 file name이라든지 state 상에서 노드에서 사용할 값이 있다면 당연히 같이 넘겨 줘야 함
- `Send`를 사용하는 노드는 그래프에 노드로 등록하지 않아도 됨 (# 2)
  - conditional edge에다가 조건 추가하듯 하면 됨 마지막 argument로 넣는 이어질 노드는 []로 감싸야됨 (# 3)
- ocr과 extractor에서는 결과가 여러 개가 나올 수 있으므로 Annotated 사용해 add를 operator로 추가 => 이렇게 하면 병렬로 처리되어 나온 결과들이 리스트에 축적됨
  - **ocr과 extractor에서 state를 업데이트 할 때도 list로 해야함**
  
 
## 마주한 오류
### Annotated 키 관련
```plain text
email attached ocr can receive only one value per step. Use and Annotated key to handle multiple values
```
- Annotated와 operator.add를 이용해 state의 타입을 축적되는 리스트가 되게 해야함
  - 이렇게 설정했는데 오류 난다면 노드의 반환 값이 리스트가 맞는지 확인해야함
  - description 지웠더니 오류 안남
