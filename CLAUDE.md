# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 현재 상태

아직 코드가 없는 기획 단계입니다. 저장소에는 `PRD.md`(요구사항), `PROMPTS.md`(4단계 개발 프롬프트), `.env`만 있습니다. 구현 전에 `PRD.md`를 먼저 읽고, 단계별 작업은 `PROMPTS.md`의 순서(뼈대 → 기능 → UI → Playwright 검토)를 따릅니다.

## 프로젝트

한국어 등 입력 텍스트를 [영어/일본어/중국어] 버튼으로 번역하는 Streamlit 웹 앱입니다. 번역은 LLM API(`openai` 라이브러리)로 처리합니다. 문서는 한국어로 작성되어 있으므로 사용자 응답과 UI 문구도 한국어를 기본으로 합니다.

## 명령어 (구현 후 기준, PRD 9~10절)

```
python -m venv .venv && .venv\Scripts\activate
pip install -r requirements.txt     # streamlit, openai, python-dotenv
streamlit run app.py                 # 로컬 실행
```

테스트/린트 도구는 아직 정해지지 않았습니다. 검증은 4단계에서 Playwright로 로컬 Streamlit URL을 열어 수행하는 방식입니다.

## 설계상 핵심 결정

- 단일 `app.py`에 UI와 `translate(text, target_language)` 함수를 두고, 번역 호출은 이 함수로 분리합니다.
- 모델은 `.env`의 `MODEL_NAME`(기본값 `gpt-6.1-sol`)로 지정하고 코드에 하드코딩하지 않습니다. 이 모델명이 실제로 유효한지는 확인되지 않았으므로, 호출 실패 시 모델명 문제부터 의심합니다.
- 설정값은 `os.getenv` → `st.secrets` 순으로 조회합니다. 로컬은 `.env`, Streamlit Community Cloud 배포는 Secrets를 쓰며, `secrets.toml`이 없어도 앱이 죽으면 안 됩니다.
- 입력 검증(빈 입력, 5,000자 제한)은 API 호출 전에 끝내고, 통과하지 못하면 호출하지 않습니다. Streamlit rerun으로 API가 중복 호출되지 않는지 주의합니다.
- 오류는 한국어 안내 메시지로 바꿔 보여주며, API 키와 스택트레이스는 화면에 노출하지 않습니다.

## 주의

- `.env`에는 실제 API 키가 들어 있을 수 있습니다. 내용을 읽거나 출력하지 말고, `.gitignore`에 포함시키며 템플릿은 `.env.example`로만 관리합니다.
