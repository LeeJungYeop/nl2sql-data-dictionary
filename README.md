# NL2SQL with Data Dictionary

> 데이터 정의서 기반 NL2SQL 성능 개선 사례 연구 정리
> 대한산업공학회 추계학술대회 2025

## 📄 Paper

**NL2SQL의 산업 현장 적용을 위한 데이터 정의서 기반 성능 개선 사례**
김대섭, 이정엽, 박민호, 김동영, 김시원, 이수동 (울산대학교)
대한산업공학회 추계학술대회, 2025. pp. 2753–2756
[DBpia](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE12484386)

## 🔍 Overview

LLM 기반 NL2SQL은 성능이 크게 향상됐지만, **사용자 질의와 DB 스키마 간 용어 불일치**로 인한 오류가 산업 현장 적용의 걸림돌이다.

본 연구는 **데이터 정의서(Data Dictionary)** 를 NL2SQL 파이프라인에 통합해, 비파괴검사(용접) 도메인의 실제 산업 데이터에서 질의 정확도를 개선한 사례를 다룬다.

**데이터 정의서**란 각 테이블·컬럼의 의미와 예시값을 정리한 문서로, DB에서 스키마를 추출한 뒤 LLM이 각 컬럼의 설명을 자동 생성하는 방식으로 구성된다. 이를 SQL 생성 프롬프트에 주입해 LLM이 사용자 질의를 정확한 스키마 요소와 연결할 수 있도록 한다.

## 📊 Results

모델: **GPT-4o** (temperature=0)
실험 데이터: 비파괴검사 도메인 자연어 질의 **24개** (하7 / 중13 / 상4)

| 난이도 | 데이터 정의서 미사용 | 데이터 정의서 사용 |
|:------:|:-------------------:|:-----------------:|
| 하     | 85.7%               | **100.0%**        |
| 중     | 53.9%               | **69.2%**         |
| 상     | 50.0%               | **75.0%**         |
| **전체** | 62.5%             | **79.2%**         |

> 전체 정확도 **+16.7%p** 향상

## 💻 적용 사례

논문의 질의 처리 파이프라인을 **LangChain** + **Chainlit** 기반 챗봇으로 구현했다. LangChain으로 SQL 생성 → DB 실행 → 답변 생성 파이프라인을 구성하고, Chainlit으로 각 단계를 사용자에게 투명하게 노출하는 챗봇 UI를 제공한다.

### 파이프라인

```
[입력] 사용자 자연어 질문
    │
    ▼
[Step 1] SQL 생성
    IN  : 사용자 질문 + 테이블 스키마 정보 + 데이터 정의서(미리 주입)
    규칙: 순수 SQL 한 문장 / 한글 컬럼명 백틱 처리 / 날짜 YYYYMMDD 형식
    OUT : SQL 문자열
    │
    ▼
[Step 2] SQL 실행
    IN  : SQL 문자열
    OUT : 쿼리 실행 결과
    │
    ▼
[Step 3] 답변 생성
    IN  : 사용자 질문 + SQL 문자열 + 쿼리 실행 결과
    규칙: 한국어 답변 / 숫자 천단위 쉼표 표기
    OUT : 최종 한국어 답변
```

도메인 지식(컬럼 의미, 값 매핑 등)은 데이터 정의서에서 일괄 관리하며, 프롬프트 자체에는 SQL 출력 형식 규칙만 명시해 역할을 분리했다.

### LangChain 파이프라인 설계

LCEL의 `RunnablePassthrough.assign()`을 체이닝해 각 단계의 출력을 컨텍스트에 누적한다. 이전 단계의 결과가 dict에 추가되는 방식으로 전달되며, 최종 답변 생성 시 모든 중간 결과를 참조할 수 있다.

```python
chain = (
    RunnablePassthrough.assign(sql=generator)
                       .assign(result=itemgetter("sql") | execute_query)
    | generate_answer
)
```

```
{ 사용자 질문 }
    → SQL 생성   → { 사용자 질문, 생성된 SQL }
    → SQL 실행   → { 사용자 질문, 생성된 SQL, 실행 결과 }
    → 답변 생성  → 최종 한국어 답변
```

## 🔧 Tech Stack

- **LLM**: GPT-4o (OpenAI)
- **Framework**: LangChain, Chainlit
- **DB**: MySQL
- **Language**: Python
