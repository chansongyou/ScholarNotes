# SWE-SEARCH: Enhancing Software Agents with Monte Carlo Tree Search and Interative Refinement

https://arxiv.org/pdf/2410.20285

## Abstract

__SWE-Search__ 를 소개.
  - Monte Carlo Tree Search와 리포지토리 레벨의 소프트웨어 작업에 대한 에이전트의 성능을 향상시키기 위한 self-improvement 메커니즘을 결합한 multi-agent framework.
  - 전통적인 방식의 MCTS을 LLM을 숫자 값 추정과 질적 평가에 활용한 하이브리드 밸류 함수를 통합해 확장.
    - Agent가 자신이 추구하는 경로에 대해 정량적 숫자 평가와 질적 자연어 평가를 바탕으로 전략을 반복적으로 개선할 수 있는 self-feedback loop를 가능하게 함.
  - Multi-Agent Framework
    - _SWE-Agent_: 적응형 탐색 담당
    - _Value Agent_: 반복적 피드백 담당
    - _Discriminator Agent_: 협력적 의사 결정을 위해 다중 에이전트의 토론을 촉진하는 역할
- MCTS를 사용하지 않았던 표준 open-source agents에 비해 __23%__의 상대적 향상을 보임

## 1. Introduction

LLM이 소프트웨어 엔지니어링 강력한 능력을 보여주고 있긴 하지만, 복잡하고 장기적은 과제를 해결하는 데는 어려움을 겪음.
  - 실제 소프트웨어 엔지니어들은 여러 솔루션을 탐색하고 피드백을 바탕으로 전략을 수정하며 가장 효휼적인 경로를 찾아 나감.
  - 반면, 현재 LLM 기반의 소프트웨어 에이전트는 강력하지만 이런 복잡하고 장기적인 과제를 해결하는 데 여전히 어려움을 겪음.

SWE-Search은 다중 에이전트 시스템으로 인간 엔지니어의 적응성, 반복 학습, 협력적 의사 결정을 복제함.
  - 소프트웨어 엔지니어링에서 세 가지 중요한 요구를 해결하도록 설계
    1. __유연한 탐색 및 적응__:
        - 엔지니어링 문제들은 종종 여러 접근법을 탐색하고 변화하는 정보에 따라 전략을 조정할 필요가 있음.
    2. __피드백을 통한 반복 학습__:
          - 효과적인 엔지니어링은 지속적인 테스트와 개선에 크게 의존. 이를 재현하기 위해 MCTS 플래닝 모듈과 Value Agent를 결합.
    3. __협력적 의사 결정__:
        - 복잡한 문제는 다양한 관점에서 보는 것이 효과적.
        - SWE-Search에서는 잠재적 해결책들이 생성되면 Discriminator Agent가 다중 에이전트 토론을 시작하고, 각 에이전트는 서로 다른 솔루션을 지지하는 주장을 하고 이를 Disciminator Agent가 비판적으로 평가.
        - 이 과정을 통해 가장 견고한 솔루션을 선택하고 정제하는 실제 엔지니어링을 반영.

SWE-Bench 벤치마크를 가지고 평가.


## 2. Related Work

### Search Method

- 가능한 선택지를 탐색하고 기억하는 전략과 이들 사이를 전환하는 휴리스틱 방식에서 차이를 보임.
  1. 너비 우선 탐색 (Breadth-first)
     - 모든 경로를 유지하지만 상당한 메모리와 계산 비용 발생.
  2. 깊이 우선 탐색 (Depth-first)
     - 상대적으로 더 greedy한 선택을 해 가장 유망한 경로를 우선시.
- LLM에 적용할 때, 이런 방법들은 텍스트 생성에서 다양성과 품질 사이의 트레이드오프가 있음을 보여줌.
- A* 알고리즘은 미리 정해진 평가 함수를 사용해 너비 우선과 탐욕적 선택을 적절히 조합해서 최적의 솔루션을 찾도록 함.
- 이 논문에서는 __Monte Carlo Tree Search__를 채용해, 각 상태에 대한 전용 평가 휴리스틱을 필요로 하지 않고 통계적 트리 탐색을 수행함.

### Software Agents

- [_SWE-Agent_](https://arxiv.org/pdf/2405.15793): Agent-Computer Interface. [repo url](https://github.com/princeton-nlp/SWE-agent)
- [_OpenDevin_](https://github.com/All-Hands-AI/OpenHands): Community-driven agents(_CodeAct_ 등)
- [_Agentless approach_](https://github.com/OpenAutoCoder/Agentless): 위치 지정 + 수정, 이 두 단계로 구성된 간단한 과정
- [_AutoCodeRover_](https://arxiv.org/pdf/2404.05427): AST 및 스펙트럼 기반의 Fault Localization
- _Alibaba Lingma Agent_: [How to Understand Whole Software Repository?](https://arxiv.org/pdf/2406.01422) 기반의 방식.

### Methodology



## 참고 자료

### System 2 Thinking

(Answered by GPT-4o)

시스템 2 사고(System 2 Thinking)는 심리학자 대니얼 카너먼(Daniel Kahneman)이 그의 저서 *생각에 관한 생각(Thinking, Fast and Slow)*에서 소개한 개념으로, 인간의 사고 과정에서 느리고 논리적이며 의식적인 사고 방식을 말합니다. 

시스템 2는 문제 해결이나 의사 결정 과정에서 주로 작동하며, 신중한 분석과 체계적인 사고가 필요할 때 활성화됩니다. 이 방식은 주어진 정보를 평가하고, 논리적 추론을 통해 해결책을 도출하거나 복잡한 문제를 해결할 때 중요한 역할을 합니다. 예를 들어, 수학 문제를 풀거나 논리적 퍼즐을 해결할 때, 시스템 2 사고가 사용됩니다.

**특징**:
- **속도**: 시스템 2는 느리게 작동하며, 시간이 필요합니다.
- **에너지 소모**: 고도로 집중된 사고를 요구하며, 많은 정신적 에너지를 소비합니다.
- **의식적 처리**: 시스템 2는 의식적인 사고를 수반하며, 정보나 규칙을 논리적으로 분석합니다.
- **복잡한 문제 해결**: 복잡하거나 새로운 문제에 직면했을 때, 비판적 사고를 적용하여 결정을 내립니다.

시스템 2 사고는 시스템 1 사고(빠르고 직관적인 사고)와는 대조적입니다. 시스템 1은 즉각적이고 자동적인 반응을 제공합니다. 둘 다 인간의 인지 시스템에서 상호작용하며 상황에 따라 적절한 사고 방식을 사용합니다. 예를 들어, 길거리에서 갑자기 차가 다가오면 시스템 1이 반사적으로 반응하는 반면, 집을 살 때처럼 복잡한 결정을 내릴 때는 시스템 2가 활성화됩니다.
