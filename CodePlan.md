# (Ongoing)CodePlan: Repository-Level Coding using LLMs and Planning

- Conference: [ACM](https://dl.acm.org/)
- PDF: https://arxiv.org/pdf/2309.12499
- Github Repo: https://github.com/microsoft/codeplan

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
- **증분 의존성 분석 (Incremental dependency analysis)**
- **변경 영향 분석 (Change may-impact)**
- **적응형 계획 알고리즘 (Adaptive planning algorithm(symbolic components))**

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

* Fig. 1에 표시된 것과 같은 **초기 사양(_seed specifications_)** 세트가 코드 편집 작업의 시작점이 됨.
* 그 초기 사양에 따라 같이 변화되어야 하는 것이 Fig. 3.에서 `process` 메서드에 필요한 편집을 **파생 사양(_derived specifications_)** 이라 함.
* 이런 모든 편집 과정이 전파돼 자동으로 전체 리포지토리를 _valid_ 한 상태로 만드는 것이 목적.
* 여기서 validity는 oracle에 의해 정의되고, 다향한 방식으로 리포지토리 레벨의 정확성을 체크:
    * 오류 없이 빌드
    * 정적 분석 통과
    * 타입 시스템 or 테스트셋 통과
    * Verification tool 통과

여기서 LLM-driven repository-level coding task는 아래와 같이 정의
```
𝑅𝑠𝑡𝑎𝑟𝑡: 리포지토리의 시작 상태
Δ𝑠𝑒𝑒𝑑𝑠: 초기 수정 사양 세트
Δ𝑑𝑒𝑟𝑖𝑣𝑒𝑑: 파생 수정 사양 세트
Θ: 오라클(oracle) 즉, Θ(𝑅𝑠𝑡𝑎𝑟𝑡)= True
𝐿: LLM 

LLM-driven repository-level coding task의 목적은 `𝑅𝑡𝑎𝑟𝑔𝑒𝑡 = 𝐸𝑥𝑒𝑐𝑢𝑡𝑒𝐸𝑑𝑖𝑡𝑠 (𝐿, 𝑅𝑠𝑡𝑎𝑟𝑡, 𝑃)` 인 𝑅𝑡𝑎𝑟𝑔𝑒𝑡에 도달하는 것인데, 
여기서 P는 초기 수정 사양 세트와 파생 사양 세트의 합인 수정 사양의 체인으로 볼 수 있음. (`Δ𝑠𝑒𝑒𝑑𝑠 ∪ Δ𝑑𝑒𝑟𝑖𝑣𝑒𝑑`)
그래서 결국 `Θ(𝑅𝑡𝑎𝑟𝑔𝑒𝑡)= True` 이 되는 것이 최종 목표.
```

### Propose Solution

리포지토리 레벨의 코딩은 계획 문제(planning problem)로 구조화해 파생 사양을 계산하는 방법을 제안
* 자동화된 계획 (Automated planning): 여러 단계의 문제 해결을 목표
* 각 단계에서 여러 대안 중 한 action을 실행해 목표 상태에 도달하도록 함.

#### CodePlan

![image](https://github.com/user-attachments/assets/95245632-904f-4a6b-bc8d-99644b71eba4)

* **CodePlan에 주어지는 것들:**
  1. 리포지토리
  2. 자연어 지시나 수동 코드 편집 세트를 통해 표현된 초기 사양(seed specifications)을 포함한 작업(task)
  3. 올바름(correctness)을 판단하는 오라클(oracle)
  4. 지시를 받아 코드를 편집할 수 있는 LLM
 
* **CodePlan이 하는 것:**
   - **계획 그래프(plan graph)** 구성.
      - 이 그래프에서 각 노드는 LLM이 해결해야 할 코드 편집 의무(code edit obligation)를 나타내고, 선(edge)은 그 다음 수행돼야 할 편집(노드)으로의 연결.
   - **코드 수정 모니터링 및 계획 그래프 확장**
      - 수정 Δ𝑠𝑒𝑒𝑑𝑠는 작업 설명으로 형성.
      - 수정 Δ𝑑𝑒𝑟𝑖𝑣𝑒𝑑는 증분 의존성 분석, 변경 영향 분석, 그리고 적응형 계획 알고리즘에 의해 형성.
   - Merge block은 LLM이 생성한 코드를 repository에 merge
   - 한 계획에서 모든 단계가 완료되면, 리포지토리는 오라클에 의해 분석.
   - 오라클이 리포지토리를 검증하면 작업 완료.
   - 에러가 발견되면, 해당 에러 리포트는 계획 생성과 실행의 반복에서 초기 사양으로써 사용됨.
 
그럼 여기서 과정을 정리해 보면:
1. Fig. 1. 과 같은 API 마이그레이션 작업이 초기 사양으로 들어옴.
2. (어디를 고칠 지 찾는 방법은 생략된 건지 나와있지 않음. 여기서 증분 의존성 분석을 통해 이루어질 듯?)
3. 1번에 나와있는 지시에 따라 코드를 수정 (여기서 Fig. 3.의 변경사항 1번만 포함)
4. 수정된 코드를 분석 후, function signature가 변경되었으므로 **_escaping change_** 로 분류
5. 변경 영향 분석으로 function signature가 변경된 해당 메서드를 호출하는 callers (Fig. 3. 에서는 `process` 메서드)를 찾음.
6. 적응형 알고리즘이 caller-callee 의존성을 이용해 영향이 미치는 다른 함수/메서드를 찾아내 파생 사양을 추론.
   - 각 사양에 따라 알맞은 프롬프트로 LLM에게 넘겨진다고 함.
7. 결과적으로 생성된 코드는 오라클을 통해 빌드 에러 없이 통과하게 됨.

이 과정이 단지 one-hop 변경 전파 과정. 실제로는 파생 변경사항은 전이적으로 다른 변경사항들을 필요로 할 수 있어서 multi-hop 과정으로 진행될 수 있음.

### Contributions



