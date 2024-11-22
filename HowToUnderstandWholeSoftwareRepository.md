# How to Understand Whole Software Repository

https://arxiv.org/pdf/2406.01422
https://github.com/LingmaTongyi/Lingma-SWE-GPT/blob/main

## Abstract & Introduction 요약

- 리포지토리를 지시 그래프로 구성
- 몬테카를로 트리 서치 기반 리포지토리 탐색 전략 이용
- 검색한 리포지토리 지식을 더 잘 사용하도록 에이전트로 요약,분석 및 계획 활용

***

## Methodology

### Overview

![image](https://github.com/user-attachments/assets/86ad548c-11de-41ce-a9e6-dc4c25d20cdb)

***

### Repository Knowledge Graph Construction

- AST를 이용해 전체 리포지토리를 Top-down 방식으로 Root(repository)에서부터 파일, 클래스, 메서드/함수 순으로 파싱해 **계층적 구조의 트리**로 구성.
- 그리고 function call 관계도를 나타내기 위해, 위에서 만든 트리 구조에서 **참조 그래프 구조**로 확장.
  - (이 논문에서는 function이 실제 코드의 수행되는 기본 유닛이기 때문에 function간의 참조만 그래프 구조에 포함. 오히려 너무 많은 관계도가 포함되면 분석 효율과 정확도에 영향을 줄 수 있다고 함.)
- 각 Entity별 메타데이터도 저장 (파일 경로, 이름, 코드 스니핏, 파일에서의 위치(line number), 등)

구조는 아래와 같음.
- Repo
  - File
    - Global var
    - Top-level function
    - Class
      - Method

***

### MCTS-Enhanced Repository Understanding

알고리즘을 통해 점점 서치 스페이스를 좁혀 나가는 접근.

Root node는 전체 리포지토리에서 시작.

총 4가지 단계가 반복
1. **Selection**
2. **Correlation expansion**
3. **Simulation & Evaluation**
4. **Backpropagation & Reference expansion**

***

#### Selection

Node selection에 사용되는 알고리즘:

![image](https://github.com/user-attachments/assets/31bb867d-2416-4041-ab0f-549ffc8951a2)

이 논문에서는 c=√2/2

***

#### Correlation Expansion

현재 Leaf 노드가 리포지토리 지식 그래프에서 child node가 있다면 확장 가능.

여기서 확장할 때 random 확장을 하기보다는 가장 연관된 child에 확장하기 위해서 여기서 두가지 방식의 확장을 이용.
1. 상호관계 확장 (Correlation expansion)
2. 참조관계 확장 (Reference relationship expansion) - 이건 backpropagation과 함께 진행되기 때문에 이 단계에서는 다루지 않음.

유저의 요구사항이나 issue description에 보통 핵심 키워드가 포함되어 있기 때문에 이것들과 연관된 코드로 확장하는 것이 유리.

그래서 유저 요구사항과 child node의 코드간의 연관성(**bm25 score**)를 계산해서 점수가 높은 child에게 확장의 우선순위를 줌.

***

#### Simulation & Evaluation

새로 확장된 노드에서 시뮬레이션을 하면서 가능한 경로들이 실제로 문제 해결에 효과적인지를 평가.

여기서도 default policy로 연관성(**bm25 score**)이 가장 높은 child 노드를 계속 선택해 나가면서 찾은 tree의 leaf 노드로 reward를 평가.

이 선택된 leaf 노드를 평가해야하는데, Reward Agent(LLM)을 활용해서 선택된 leaf 노드의 연관성을 평가.
- Reward Agent: In-context learning(ICL) + Chain-of-Thought(CoT) 활용.
- leaf 노드와 problem description을 프롬프트에 포함해 LLM에게 요청하면 설명+점수를 받음.
  - 여기서 주는 점수가 Reward가 됨.
  - 점수가 6 이상인 노드는 유지(?).
 
**이 과정에서의 의문점**
1.  시뮬레이션에서 연관성 즉 bm25 점수가 가장 높은 Child를 지식 그래프의 리프 노드까지 계속해서 선택하는 걸로 이해를 했는데, 그럼 1개만 선택이 되어야 하는데, ~논문에서는 여러개라고 말하는 듯한 느낌~.
  - ~그렇다면 여러 경로를 탐색하고 각 경로에 점수를 매겨서 6점 이상인 경로만 남기는 건지..? 그럼 확장된 노드에 대한 Reward는 뭘로 정해주는 건지..?~
  - 아니면, 혹시 점수가 6점 이상이면 이 시뮬레이션을 거쳐온 경로도 다 expansion 된걸로 유지시키는? 아래의 reference expansion 부분을 읽다보면 이쪽이 좀 더 가능성 있음.
  - **해결**: 후자가 정답.

***

#### Backpropagation & Reference Expansion

시뮬레이션과 평가가 끝나면 terminal node에서부터 bottom-up으로 업데이트. 이건 기존의 MCTS와 동일.

Reward가 6이상인 terminal 노드에서는 참조 확장이 일어남. 여기서 호출하는 함수들을 리포지토리 지식 그래프에 기반해서 확장.
- **의문점**: 여기서도 그러면 (함수는 이미 leaf노드일 것이기 때문에 시뮬레이션 필요X) evaluation + backpropagation 이 트리거 되는건지..? 이런 것에 대한 자세한 설명은 나와있지 않음.
- 참조관계 확장, 평가, backpropagation이 진행이 된다면, 참조된 함수의 부모 노드들도 자연적으로 자연적으로 추가가 될 듯?
- **해결**: 코드를 보니, reference expansion은 local_expand 로 구현됨. 예상한 대로 참조 확장으로 추가되는 노드의 부모 노드들이 모두 MCTS의 트리에 추가됨. 그리고 evaluation과 backpropagation이 발생.

***

### Information Utilization & Patch Generation

다음과 같은 순서로 진행
1. 전체 리포지토리 경험을 먼저 요약
2. 이를 기반으로 동적으로 필요한 코드 스니핏 정보 획득
3. 문제를 해결할 패치 생성

***

#### Repository Summary

1. MCTS를 이용해 문제/유저요구사항과 연관된 contents를 수집 후, summary agent(LLM)에게 요청.
2. 위에서 제공한 contents 전체를 넣어주기엔 너무 크기 때문에, 해당 contents의 위치와 summary agent의 output만 RepoUnderstander에게 전달.
   - 위치 예시: <file>a.py</file><class>ClassA</class><func>func a</func>

***

#### Dynamic Information Acquisition

Repository summary에서 전달받은 정보를 바탕으로 문제와 현재 워크스페이스의 이해를 도와, 빠르게 솔루션을 찾아나감.

여기서 RepoUnderstander가 추가로 필요한 정보들을 툴을 이용해 동적으로 수집.
- ReAct 방식 채용.
- 이 논문에서는 AutoCodeRover의 search API method를 따름. (search_class, search_method, search_code)
- 간략하게 AutoCodeRover이 하는 방식을 나열하면,
  1. Search API 호출이 필요한지를 결정.
  2. 리포지토리 지식 그래프에서 retrieval API를 이용해 관련 classes, methods와 코드스니핏을 검색.
  3. 찾은 결과를 반환.

***

#### Patch Generation

1. 수정이 필요한 코드 추출
2. 수정된 코드 생성 by LLM
3. 두 코드로 diff 추출
4. 적용 가능한 diff가 나올때까지 재시도. (여기서는 max=3)

***

## Experiment

### SWE-bench-lite 비교

![image](https://github.com/user-attachments/assets/74665d9d-cdd3-4112-af74-ee267487f5bc)

**Configurations**
- LLM Model for Agent-based: GPT-4-Turbo(gpt-4-1106-preview)
- MCTS:
  - max_iterations = 600
  - maximum_search_time = 300s
- Summary Agent:
  - Top-k = 10 (10개의 관련 코드 스니핏을 요약하는데 사용)

***

### MCTS, Summary Agent의 유무 비교

![image](https://github.com/user-attachments/assets/fd816075-2683-4384-be6e-ba09442a6e08)

- w.review: review agent를 추가해서 생성된 patch에 대해 review해서 실패하면 다시 생성하도록 해서 이 과정을 최대 3번까지 진행한 경우.

***

### Hyperparameter 비교

![image](https://github.com/user-attachments/assets/5f1b03b1-d512-422e-9102-c644cc6e5830)

***

### SWE-agent와 결과 분포 비교

![image](https://github.com/user-attachments/assets/8b3fc3dc-f3db-456c-90d4-2f79ab767b49)

- 제대로된 패치를 생성하는 부분에서는 swe-agent 보다 부족하지만, localization에서는 좋은 성능을 보여줌.

#### 문제 해결 과정 상세 비교

![image](https://github.com/user-attachments/assets/20cfd00c-9138-435d-aa08-35db80fc4d43)

- MCTS 의 경우엔 최종적으로 패치 생성에서 실패하더라도 추론 과정을 정확하게 판단하는 모습을 보여줌.

