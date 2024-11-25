# RepoAgent: An LLM-Powered Open-Source Framework for Repository-level Code Documentation Generation

- Conference: [EMNLP](https://2024.emnlp.org/)
- PDF: https://arxiv.org/pdf/2402.16667
- GitHub Repo: https://github.com/OpenBMB/RepoAgent

## Abstract & Introduction 요약

- LLM을 이용한 코드 도큐먼트 생성, 유지 및 업데이트.
- Git에서 변경사항을 자동으로 감지 후 문서화.
- 기존 자동 문서화 도구들은 코드 간의 의존성을 다루지 못하거나 코드 변경에 따른 업데이트가 부족.

## 2. RepoAgent

**3 key stages**

1. **Global structure analysis**
2. **Document generation**
3. Document update

### 2.1 Global Structur Analysis

1. **AST** 분석으로 파일 안에 있는 모든 클래스/함수 파싱. (메타데이터도 함께 저장)
2. 분석한 내용을 **프로젝트 트리** 형식으로 구성.
  - 루트 노드 = 전체 리포지토리
  - 중간 노드 = 디렉토리, 파일
  - 리프 노드 = 클래스, 함수 (물론 중간 노드도 될 수 있음)
3. 그리고 **LSP(Jedi)** 를 이용해 함수 간 참조 관계를 caller에서 callee로 edge를 연결해 프로젝트 트리를 **DAG 구조**로 확장.

### 2.2 Document Generation

프롬프트의 구성요소
1. 프로젝트 트리
2. 해당 코드 스니핏
3. 참조 관계도
4. 메타 정보(타입, 파일 이름, 경로 등)
5. 도큐먼트 포맷 가이드

프로젝트 트리 예시:
```text
Currently, you are in a project, and the hierarchical structure of this project is as follows:
autogpts
  autogpt
    autogpt
      commands
        user_interaction.py
          ask_user
            *ask_user
```

메타 정보(경로) 예시:
```text
The path of the document you need to generate in this project is:
autogpts/autogpt/autogpt/commands/user_interaction.py/ask_user.
```

코드 스니핏 예시
```text
Now you need to generate a document for a Function, whose name is "ask user".

The content of the code is as follows:

async def ask_user(question: str, agent: Agent) -> str:
  print(f"\nQ: {question}")
  resp = await clean_input(agent.legacy_config, "A:")
  return f"The user’s answer: ’{resp}’"
```

참조 관계도 예시
```
As you can see, the code calls the following objects, their code and docs are as
following:

OBJ NAME: clean_input
OBJ PATH: autogpts/autogpt/autogpt/app/utils.py/clean_input

Document:
**Function Name**: clean_input
(도큐먼트 내용 생략)

[Code begin of clean input]
(코드 생략)
[Code end of clean input]

... (다른 참조 함수 생략)
```

### Document Update

- pre-commit hook을 이용해 코드 변경을 감지하고 문서 업데이트.

업데이트가 트리거 되는 경우
1. 소스코드 수정 감지
2. 객체의 참조자가 더 이상 해당 객체를 참조하지 않을 때
3. 객체가 새로운 참조를 받을 떄

업데이트가 트리거 되지 않는 경우
1. A함수가 B함수를 참조하고 있을 때 B함수가 바뀌더라도 A함수의 문서를 업데이트 하지 않음.

## Experiments

사용한 모델
- gpt-3.5-turbo
- gpt-4-0125
- Llama-2-7b
- Llama-2-70b

### Human Evaluation

- 사람이 만든 문서 vs LLM이 생성한 문서 비교.
- Transformer와 Llama-Index 리포지토리의 150개의 임의의 샘플(100개의 클래스, 50개의 함수)로 진행.
- 3명의 평가자를 고용해서 문서의 선호도 비교
- (여기선 어떤 모델로 만든 문서라는 건 나와있지 않음)

![image](https://github.com/user-attachments/assets/acd5486c-46b3-4970-a392-454b1a3b6599)

## 이하 생략

