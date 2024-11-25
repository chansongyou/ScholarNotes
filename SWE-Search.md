# (Ongoing)SWE-Search: Enhancing Software Agents with Monte Carlo Tree Search and Interative Refinement

https://arxiv.org/pdf/2410.20285

## Abstract

**SWE-Search** 를 소개.
  - Monte Carlo Tree Search(MCTS)와 리포지토리 레벨의 소프트웨어 작업에 대한 에이전트의 성능을 향상시키기 위한 self-improvement 메커니즘을 결합한 multi-agent framework.
  - 전통적인 방식의 MCTS을 LLM을 숫자 값 추정과 질적 평가에 활용한 하이브리드 밸류 함수를 통합해 확장.
    - Agent가 자신이 추구하는 경로에 대해 정량적 숫자 평가와 질적 자연어 평가를 바탕으로 전략을 반복적으로 개선할 수 있는 self-feedback loop를 가능하게 함.
  - Multi-Agent Framework
    - _SWE-Agent_: 적응형 탐색 담당
    - _Value Agent_: 반복적 피드백 담당
    - _Discriminator Agent_: 협력적 의사 결정을 위해 다중 에이전트의 토론을 촉진하는 역할
- MCTS를 사용하지 않던 표준 open-source agents에 비해 __23%__의 상대적 향상을 보임

## 1. Introduction

LLM이 소프트웨어 엔지니어링 강력한 능력을 보여주고 있긴 하지만, 복잡하고 장기적은 과제를 해결하는 데는 어려움을 겪음.
  - 실제 소프트웨어 엔지니어들은 여러 솔루션을 탐색하고 피드백을 바탕으로 전략을 수정하며 가장 효휼적인 경로를 찾아 나감.
  - 반면, 현재 LLM 기반의 소프트웨어 에이전트는 강력하지만 이런 복잡하고 장기적인 과제를 해결하는 데 여전히 어려움을 겪음.

SWE-Search은 다중 에이전트 시스템으로 인간 엔지니어의 적응성, 반복 학습, 협력적 의사 결정을 복제함.
  - 소프트웨어 엔지니어링에서 세 가지 중요한 요구를 해결하도록 설계
    1. **유연한 탐색 및 적응**:
        - 엔지니어링 문제들은 종종 여러 접근법을 탐색하고 변화하는 정보에 따라 전략을 조정할 필요가 있음.
    2. **피드백을 통한 반복 학습**:
          - 효과적인 엔지니어링은 지속적인 테스트와 개선에 크게 의존. 이를 재현하기 위해 MCTS 플래닝 모듈과 Value Agent를 결합.
    3. **협력적 의사 결정**:
        - 복잡한 문제는 다양한 관점에서 보는 것이 효과적.
        - SWE-Search에서는 잠재적 해결책들이 생성되면 Discriminator Agent가 다중 에이전트 토론을 시작하고, 각 에이전트는 서로 다른 솔루션을 지지하는 주장을 하고 이를 Disciminator Agent가 비판적으로 평가.
        - 이 과정을 통해 가장 견고한 솔루션을 선택하고 정제하는 실제 엔지니어링을 반영.

**SWE-Bench 벤치마크를 가지고 평가.**


## 2. Related Work

### Search Method

- 가능한 선택지를 탐색하고 기억하는 전략과 이들 사이를 전환하는 휴리스틱 방식에서 차이를 보임.
  1. 너비 우선 탐색 (Breadth-first)
     - 모든 경로를 유지하지만 상당한 메모리와 계산 비용 발생.
  2. 깊이 우선 탐색 (Depth-first)
     - 상대적으로 더 greedy한 선택을 해 가장 유망한 경로를 우선시.
- LLM에 적용할 때, 이런 방법들은 텍스트 생성에서 다양성과 품질 사이의 트레이드오프가 있음을 보여줌.
- A* 알고리즘은 미리 정해진 평가 함수를 사용해 너비 우선과 탐욕적 선택을 적절히 조합해서 최적의 솔루션을 찾도록 함.
- 이 논문에서는 __MCTS__를 채용해, 각 상태에 대한 전용 평가 휴리스틱을 필요로 하지 않고 통계적 트리 탐색을 수행함.

### Software Agents

- [_SWE-Agent_](https://arxiv.org/pdf/2405.15793): Agent-Computer Interface. [repo url](https://github.com/princeton-nlp/SWE-agent)
- [_OpenDevin_](https://github.com/All-Hands-AI/OpenHands): Community-driven agents(_CodeAct_ 등)
- [_Agentless approach_](https://github.com/OpenAutoCoder/Agentless): 위치 지정 + 수정, 이 두 단계로 구성된 간단한 과정
- [_AutoCodeRover_](https://arxiv.org/pdf/2404.05427): AST 및 스펙트럼 기반의 Fault Localization.
  - **TODO** 스펙트럼 기반의 Fault Localization이 뭔지 나중에 해당 논문을 확인해봐야 할듯.
- _Alibaba Lingma Agent_: [How to Understand Whole Software Repository?](https://arxiv.org/pdf/2406.01422) 기반의 방식.

## 3. Methodology

SWE-Search는 동적 플래닝, 가치 추정, 그리고 신중한 의사 결정을 결합해 복잡한 소프트웨어 엔지니어링 업무를 처리하도록 디자인 된 다중 에이전트 시스템.
이 방법을 고안하게 된 가장 큰 동기는 인간 소프트웨어 엔지니어의 복잡하고 반복적인 워크플로우를 모방하기 위해서. 탐색하고, 계획하고, 그리고 협력하는 부분이 이런 복잡한 문제를 해결하는데 중요.
- MCTS → Planning
- Value Agent → 유틸리티 추정 및 피드백
- Discriminator → 토론을 통한 최종 의사 결정

__For Primary Components__
 1. __SWE-Search Framework and Action Agent__: Moatless-tools 프레임워크를 기반으로 구축. Git과 같은 커밋 트리 구조를 갖춘 동적 코드 환경에서 작동. 이전 상태로 효율적으로 백트래킹이 가능해서 다양한 해결 경로 탐색.
 2. __Search Algorithm__: 핵심인 __MCTS__를 채용한 탐색 전략. AlphaZero와 비슷한 휴리스틱 기반의 선택 프로세스를 이용. MCTS 알고리즘을 소프트웨어 엔지니어링 업무용으로 수정.
 3. __Value(Function) Agent__: 각 Observation의 유틸리티를 측정하기 위한 LLM-based value function. 자연어로 된 설명과 숫자를 반환함. 여기서 나온 설명은 그 다음 액션을 향상시키기 위해 사용됨(Self-feedback loop).
 4. __Discriminator Agent__: 마지막 단계에서 검색 과정에서 생성된 솔루션을 평가. 여기서 멀티 에이전트가 토론 후 최종 결정.

### 3.1 Problem Formulation

SWE-Agent의 작업 M

- `M = (S, C, A, V, P, p_0, ρ)`

- S: 상태 공간 (State Space). 에이전트가 작업 중인 파일의 현재 컨텍스트와 코드베이스의 전체 상태와 같은 모든 가능한 상태를 포함.

- C: 컨텍스트 공간(Context Space). 리포지토리와 초기 문제 설명과 같은 메타데이터.

- V: 가치 함수(Value Function). 각 상태-행동 페어인 O(a, t)에 대해 유틸리티 점수를 할당해서 에이전트의 결정에 영향. 

- P: 컨텍스트에 따라 달라지는 전이 함수(Context-dependent Transition Function).
  - `S × A × C → ∆(S)`
  - 각 행동 후에 리포지토리 상태의 변화를 나타냄.
  - 여기서 A는 Action으로 유추.

- p_0: 초기 상태 분포(Initial State Distribution).
  - `C → ∆(S)`
  - 주어진 Context에 따라 초기 상태가 어떻게 결정되는지를 나타냄.

- ρ: Context에 대한 분포
  - `ρ ∈ ∆(C)`

주어진 초기 컨텍스트 c\~ρ와 초기 상태로 c_0\~p_0(·|c) SWE-Agent가 policy π를 수행.

- π: `S x C → ∆(A)`. 현재 상태와 컨텍스트를 기반으로 액션을 선택

매 스텝 t마다, 에이전트는 액션 a_t ~ π(s_t, c)를 취하고, 그에 따른 보상 R(s_t, a_t, c)를 받음.

그리고 환경이 새로운 상태인 s_{t+1} ~ P(·|s_t, a_t, c)로 전이되고, 에이전트는 이 업데이트 된 상태를 확인.

시간이 지나면서, Agent가 환경과 상호작용 하면서 이 과정이 반복되어 궤적(Trajectory) τ를 생성.
 - ![image](https://github.com/user-attachments/assets/64bb3083-08ef-4efa-8f37-2e1442cc1512)

**에이전트의 목적은 궤적에 걸쳐 누적 보상을 최대화 하는 것.**

여기서 보상을 정하는 가치 함수 ![image](https://github.com/user-attachments/assets/99e55366-31f9-465d-bf67-64e4e3875ca3)
 - 이 가치 함수는 현재 상태와 행동뿐만 아니라 이전 상태와 행동의 역사에도 의존하기 때문에 마르코프 과정의 가정에서 벗어남.

![image](https://github.com/user-attachments/assets/3f161e96-e1e0-49d2-bcd0-1e3cce9553ab)

---

### 3.2 SWE-Search Framework and Action Agent

#### SWE-SEARCH 프레임워크 개요

- **moatless-tools 프레임워크**를 기반으로 구축.
- 소스 코드 저장소의 상태를 다룰 수 있는 유연한 상태 공간과 **git과 유사한 커밋 트리 구조**를 활용해 이전 상태로 돌아가는 등 다양한 경로를 탐색하 수 있음.

**계층적 구조**: 작업 공간은 두 계층으로 구성되어 있으며, 작업 유형(Action Type)과 해당 유형에 따라 실행 가능한 작업(Action)으로 나뉨.
  - **작업 유형(T)**: `검색(Search)`, `계획(Plan)`, `편집(Edit)`
  - **작업(Action)**: 각 유형별로 수행할 수 있는 구체적인 작업(예: 특정 코드를 편집하거나 테스트를 실행).

**상태 전환의 유연성**
- 기본적인 상태 전환 규칙은 `Plan → Edit`과 같은 고정된 방식으로 제한될 수 있지만, SWE-Search의 액션 에이전트는 이러한 제한을 완화하여 **모든 상태 간 자유로운 전환**이 가능하도록 설계.

**테스트 실행과 활용**
- 액션 에이전트는 코드베이스 내에서 실행 가능한 **테스트를 실행하거나 새로운 테스트를 생성**할 수 있음.
- 테스트 결과는 가치 함수(Value Function)에 통합되어 이후 의사 결정에 반영.
- 저장소에 존재하는 기존 테스트를 활용해 작업을 수행.

### 3.3 Value(Function) Agent

수식도 있고 조금 복잡할 수 있지만, 간단하게 정리하자면,
1. 전체 궤적(현재까지의 actions) + 현재의 상태-액션 쌍에 대해 분석.
2. 유효성 측정 값 v_t(Reward)와 그에 대한 설명 ε_t를 제공.
  - (v_t는 MCTS에서 선택하는 부분에 영향을 주고 ε_t 또한 추후 액션에 영향을 주는 듯)

### 3.4 Search Algorithm

- **MCTS(몬테 카를로 트리 서치)** 트리 활용.
  - Node = 상태 S_t
  - Edge = 행동 A_t
  
- 변형된 UCT 알고리즘 사용.
  - 서치 프로세스에서 일찍 시행된 탐험에 보너스를 주고, 나중 스테이지에서 탐험된 경우 페널티를 줌
  - ![image](https://github.com/user-attachments/assets/1636104d-a853-4e65-98d9-a63e7025a771)
  - 기존의 UCT에서 node의 depth에 따라서 값이 추가됨.

지금까지의 설명을 토대로 진행 과정을 정리해보면 아래의 과정 반복:
1. 변형된 UCT를 이용해 선택/확장 결정
2. ~~기존 MCTS처럼 action에 의해 node가 확장되면 그 이후 쭉 trajectory 진행 후,~~ 현재까지의 trajectory와 현재 액션에 대해 value agent가 reward 계산.
3. 나서 확장된 노드에 reward 반영
4. Backpropagation

#### 3.4.1 Discriminator Agent

SWE-Search의 마지막 단계.
- 최대 5개의 최종 솔루션을 가지고 여러 에이전트에게 토론을 하도록 함. 각 에이전트는 각 솔루션을 지지하도록 진행.
- Discriminator는 여러 에이전트의 의견을 수집해서 최종 선택.

## 4. Experiments

- Benchmark: SWE-bench Lite (300 instances)
- Evaluation Metrics: Pass@1 / Pass@5
- SWE-Search에서 이용한 값:
  - max_expansions = 3
  - max_iterations = 100

### 4.1 Results

![image](https://github.com/user-attachments/assets/37524d28-544c-4a6a-b62e-90f64679a412)

- 평균적으로 약 23% 성능 향상. (몇 번을 수행해서 평균을 냈는 지는 나오지 않음)
- 여기서 Moatless-adapted는 검색->식별->수정 이 3가지 순서를 따르기 보다는 자유롭게 전환될 수 있도록 바꾼 방식.
  - Moatless-v1과 Moatless-adapted 간의 차이는 약 1.4%로 그렇게 크지 않음.
 
#### Impact of Hindsight Feedback on Agent Performance (다시 돌아볼 때 쓸 피드백이 에이전트의 성능에 주는 영향)

![image](https://github.com/user-attachments/assets/7e61c88f-49e6-4542-b766-105c6d13185c)

피드백이 사용되는 방법.
1. 맨 왼쪽의 Edit 노드에서 회색의 Finish 노드로 확장 후 Value agent가 평가.
2. 두 번 째 아래의 하늘색 Edit 노드가 확장될 때, 프롬프트에 낮은 reward를 받았던 피드백을 추가해줌으로써 다른 방향으로 확장될 수 있도록 도움.

#### 4.2 Importance of Comprehensive State Information for Value Function Performance (포괄적 상태 정보가 Value function의 성능에 미치는 영향)

**포괄적 상태 정보란?**
- 상태 정보: 
- 각 상태(검색, 계획, 수정 등)별로 프롬프팅을 달리 해야함.



## 사용된 하이퍼파라미터 값들

![image](https://github.com/user-attachments/assets/4f078c6a-8ae7-4704-9697-fe95730d6e90)

  

   
