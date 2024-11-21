# CodePlan: Repository-Level Coding using LLMs and Planning

https://arxiv.org/pdf/2309.12499

https://github.com/microsoft/codeplan

## Abstract

Repository 레벨의 코딩 작업을 아래와 같이 나열:
1. 패키지 마이그레이션
2. Static analysis나 testing을 이용한 에러 리포트 해결
3. 타입 어노테이션 추가
4. 코드베이스데 대한 여러 요구사항들 (전체 코드 리포지토리에 대한 수정 등)

GitHub Copilot처럼 LLMs을 활용한 툴은 지역화된 코딩 문제를 해결하는데는 뛰어나지만 Repository 레벨의 코딩 작업은 코드 내에서 상호 의존적으로 얽혀있고 프롬프트에 다 넣기엔 너무 커서 어려움.

CodePlan는 multi-step chain-of-edits (plan)으로 매 단계에서 LLM을 호출하는데 다음과 같은 정보들을 제공:
- 전체 리포지토리에 연관된 컨텍스트
- 코드의 위치
- 이전의 코드 체인지
- 작업에 대한 구체적인 지침

CodePlan은 다음과 같은 과정들을 포함:
- 점진적 의존성 분석 (Incremental dependency analysis)
- 변경 영향 분석 (Change may-impact)
- 적응형 계획 알고리즘 (Adaptive planning algorithm(symbolic components))

다음 두 repository 레벨의 작업으로 비교를 진행:
1. 패키지 마이그레이션 (C#)
2. 시간적 코드 수정 (Python)

두 작업 종류 모두 여러 파일에 걸쳐 상호 의존적인 코드들의 수정을 요함 (2~97개의 파일)

## 1. Introduction

만약 기존에 사용하고 있던 라이브러리 _L_ 이 버전 v_n에서 v_n+1 로 넘어가면서 API에 변화가 있었다면, 
우리 리포지토리에서 다음 변경사항들을 모두 적용해야 하는 피키지 마이그레이션이 필요하게 되는데, 이런 경우엔 라이브러리 _L_ 을 이용하고 있는 모든 부분에서 수정이 필요할 수 있음.

예로는 아래와 같은 상황이 있다.
![image](https://github.com/user-attachments/assets/14902c4c-0220-4d89-a5e0-4ab90196c454)

그럼 결과적으로 아래와 같은 변경사항이 필요하게 됨.
![image](https://github.com/user-attachments/assets/06232ae1-7632-4e97-a05d-c2a65f49b392)

### Problem Formulation

* Fig. 1에 표시된 것과 같은 **초기 사양(_seed specifications_)** 이 코드 편집 작업의 시작점이 됨.
* 그 초기 사양에 따라 같이 변화되어야 하는 것이 Fig. 3.에서 `process` 메서드에 필요한 편집을 **파생 사양(_derived specifications_)** 이라 함.
* 이런 모든 편집 과정이 전파돼 자동으로 전체 리포지토리를 _valid_ 한 상태로 만드는 것이 목적.
* 여기서 validity는 oracle에 의해 정의되고, 다향한 방식으로 리포지토리 레벨의 정확성을 체크:
    * 오류 없이 빌드
    * 정적 분석 통과
    * 타입 시스템 or 테스트셋 통과
    * Veritication tool 통과

