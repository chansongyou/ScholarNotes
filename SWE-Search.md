# SWE-SEARCH: Enhancing Software Agents with Monte Carlo Tree Search and Interative Refinement

https://arxiv.org/pdf/2410.20285

## Abstract

__SWE-Search__ 를 소개.
  - Monte Carlo Tree Search와 리포지토리 레벨의 소프트웨어 작업에 대한 에이전트의 성능을 향상시키기 위한 self-improvement 메커니즘을 결합한 multi-agent framework.
  - 전통적인 방식의 MCTS을 LLM을 숫자 값 추정과 질적 평가에 활용한 하이브리드 밸류 함수를 통합해 확장.
    - Agent가 자신이 추구하는 경로에 대해 정량적 숫자 평가와 질적 자연어 평가를 바탕으로 전략을 반복적으로 개선할 수 있는 self-feedback loop를 가능하게 함.
  - Multi-Agent Framework
    - _SWE-Agent_: 적응형 탐색 담당
    - _Valude Agent_: 반복적 피드백 담당
    - _Discriminator Agent_: 협력적 의사 결정을 위해 다중 에이전트의 토론을 촉진하는 역할
- MCTS를 사용하지 않았던 표준 open-source agents에 비해 __23%__의 상대적 향상을 보임
