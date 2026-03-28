# NL2SQL with Data Dictionary

> 데이터 정의서를 프롬프트에 주입해 산업 현장 NL2SQL 정확도를 +16.7%p 개선

![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=flat&logo=langchain&logoColor=white)
![OpenAI](https://img.shields.io/badge/GPT--4o-412991?style=flat&logo=openai&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white)

---

## Results

GPT-4o 기반, 비파괴검사(용접) 도메인 자연어 질의 24개 실험

| 난이도 | 미적용 | 적용 | 향상 |
|:------:|:------:|:----:|:----:|
| 하 | 85.7% | **100.0%** | +14.3%p |
| 중 | 53.9% | **69.2%** | +15.3%p |
| 상 | 50.0% | **75.0%** | +25.0%p |
| **전체** | 62.5% | **79.2%** | **+16.7%p** |

---

## Problem

LLM 기반 NL2SQL의 실제 적용을 막는 주요 원인은 **사용자 질의 ↔ DB 컬럼명 간 용어 불일치**다.

예시: 사용자는 "불합격"이라고 질문하지만 DB에는 "부적합"으로 저장됨 → SQL 생성 실패

---

## Solution

스키마에서 자동으로 **데이터 정의서**를 생성하고 SQL 생성 프롬프트에 주입.

```
[데이터 정의서 구조]
컬럼명 | 타입 | 예시값 | 설명
─────────────────────────────────────────────
최종 판정 | VARCHAR | 합격, 부적합 | "불합격" 아닌 "부적합" 사용
결함 유형  | VARCHAR | CR, PO, LF   | CR=Crack, PO=Porosity
촬영일자   | INT     | 20220301     | YYYYMMDD 형식 숫자로 저장
```

---

## Pipeline

```
사용자 자연어 질문
    │
    ▼
[Step 1] SQL 생성   ←  스키마 + 데이터 정의서 주입
    │
    ▼
[Step 2] SQL 실행   →  MySQL 쿼리 실행
    │
    ▼
[Step 3] 답변 생성  →  한국어 자연어 답변
```

LangChain LCEL `RunnablePassthrough.assign()` 체이닝으로 각 단계 결과를 컨텍스트에 누적.

---

## Tech Stack

| 분류 | 기술 |
|------|------|
| LLM | GPT-4o (temperature=0) |
| Framework | LangChain, Chainlit |
| DB | MySQL |
| Language | Python |

---

## Publication

대한산업공학회 추계학술대회 2025, pp. 2753–2756
김대섭, **이정엽**, 박민호, 김동영, 김시원, 이수동 (울산대학교)
[DBpia](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE12484386)
