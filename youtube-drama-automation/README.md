# 🎭 YouTube → 일본 와비사비 심리 드라마 자동 제작 시스템

YouTube 콘텐츠를 일본 와비사비 스타일의 심리 드라마로 자동 변환하는 Make.com 기반 자동화 시스템입니다.

## 🎯 프로젝트 개요

### 핵심 흐름
```
YouTube URL → RAW 대본 추출 → 와비사비 스타일 각색 → SEO 메타데이터 생성 → Google Sheets 저장
```

### 주요 기능
- ✅ YouTube Transcript API 자동 호출
- ✅ LLM 기반 와비사비 스타일 각색 (커스텀 태그 포함)
- ✅ SEO 최적화 메타데이터 자동 생성
- ✅ Google Sheets 자동 저장
- ✅ Telegram 알림 (선택)

## 📁 프로젝트 구조

```
youtube-drama-automation/
├── README.md                          # 이 파일
├── make-scenarios/                    # Make.com 시나리오
│   ├── blueprint.json                 # Make.com import용 시나리오
│   └── README.md                      # 시나리오 설명
├── prompts/                           # LLM 프롬프트 템플릿
│   ├── wabisabi_script_prompt.txt     # 와비사비 각색용
│   └── seo_generation_prompt.txt      # SEO 생성용
├── schemas/                           # 데이터 스키마
│   ├── google_sheets_schema.json      # Google Sheets 구조
│   └── youtube_input_schema.json      # 입력 데이터 스키마
├── api-templates/                     # API 요청 템플릿
│   ├── youtube_transcript_request.json
│   └── llm_request_template.json
├── examples/                          # 예제 파일
│   ├── sample_raw_transcript.txt
│   └── sample_wabisabi_output.txt
└── docs/                              # 문서
    ├── setup_guide.md                 # 설정 가이드
    └── automation_flow.md             # 자동화 플로우 설명
```

## 🪶 와비사비 스타일 규칙

### 핵심 원칙
- **대상**: 일본 시니어(50대~60대) 취향
- **분위기**: 정적·절제·여백·고요
- **테마**: 인과응보, 고독, 잔잔한 복수, 고요한 해방
- **금기**: 선정성, 직접적 폭력 묘사

### 커스텀 태그 시스템
```
[TTS_TAG: 감정_강약]           # TTS 감정 제어
[AUDIO_CUE: BGM_종류]          # 배경음악 지시
[시간_TAG: 침묵_3초]           # 시간/침묵 제어
[SCENE: 장면_전환]             # 장면 전환
```

## 🚀 빠른 시작

### 1. Make.com 시나리오 설정
```bash
# make-scenarios/blueprint.json을 Make.com에 import
```

### 2. Google Sheets 준비
```bash
# schemas/google_sheets_schema.json을 참고하여 시트 생성
```

### 3. API 설정
- YouTube Transcript API 키 발급
- LLM API (OpenAI/Anthropic/etc.) 키 발급
- Make.com에서 API 연결

### 4. 자동화 실행
- Google Sheets에 YouTube URL 입력
- Make.com 시나리오 실행
- 결과 확인

## 📖 상세 문서

- [설정 가이드](./docs/setup_guide.md)
- [자동화 플로우](./docs/automation_flow.md)
- [Make.com 시나리오](./make-scenarios/README.md)

## 🔧 기술 스택

- **자동화**: Make.com
- **데이터**: Google Sheets
- **API**: YouTube Transcript API, LLM API
- **스타일**: 일본 와비사비 미학

## 📝 라이선스

이 프로젝트는 개인 프로젝트이며, 상업적 사용 시 별도 협의가 필요합니다.

---

**제작**: Claude Code 자동 생성
**업데이트**: 2025-11-23
