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
