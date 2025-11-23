# 🚀 YouTube 와비사비 드라마 자동화 설정 가이드

이 가이드는 Make.com 기반 YouTube → 와비사비 드라마 자동화 시스템을 처음부터 설정하는 방법을 설명합니다.

## 📋 목차

1. [사전 준비사항](#사전-준비사항)
2. [Google Sheets 설정](#google-sheets-설정)
3. [API 키 발급](#api-키-발급)
4. [Make.com 시나리오 설정](#makecom-시나리오-설정)
5. [프롬프트 파일 업로드](#프롬프트-파일-업로드)
6. [테스트 실행](#테스트-실행)
7. [문제 해결](#문제-해결)

---

## 사전 준비사항

### 필수 계정
- ✅ Google 계정 (Google Sheets 사용)
- ✅ Make.com 계정 (무료 플랜 가능)
- ✅ OpenAI 계정 (API 키 발급용)
- ✅ Telegram 계정 (알림용, 선택사항)

### 필수 파일
이 프로젝트의 다음 파일들을 준비하세요:
- `make-scenarios/blueprint.json`
- `prompts/wabisabi_script_prompt.txt`
- `prompts/seo_generation_prompt.txt`
- `schemas/google_sheets_schema.json`

---

## Google Sheets 설정

### 1단계: 새 스프레드시트 생성

1. [Google Sheets](https://sheets.google.com) 접속
2. **빈 스프레드시트** 생성
3. 스프레드시트 이름: `YouTube 와비사비 드라마 자동화`

### 2단계: 시트 3개 생성

#### 시트 1: `대기중_영상`
헤더 행에 다음 항목 추가:

| row_id | youtube_url | status | added_at | notes |
|--------|-------------|--------|----------|-------|
| 1 | https://www.youtube.com/watch?v=example | 대기중 | 2025-11-23 10:00:00 | 테스트 |

#### 시트 2: `완성_대본`
헤더 행에 다음 항목 추가:

```
row_id | youtube_url | raw_transcript | wabisabi_script | title | subtitle |
short_description | full_description | tags | keywords | hashtags |
category | target_audience | emotional_hook | status | created_at
```

#### 시트 3: `오류_로그`
헤더 행에 다음 항목 추가:

```
row_id | youtube_url | error_message | error_step | error_time | retry_count
```

### 3단계: Spreadsheet ID 복사

1. 스프레드시트 URL 확인:
   ```
   https://docs.google.com/spreadsheets/d/SPREADSHEET_ID/edit
   ```
2. `SPREADSHEET_ID` 부분을 복사하여 메모장에 저장

---

## API 키 발급

### OpenAI API 키

1. [OpenAI Platform](https://platform.openai.com) 접속
2. **API Keys** 메뉴로 이동
3. **Create new secret key** 클릭
4. 키 이름 입력 (예: `wabisabi-automation`)
5. 생성된 키를 안전한 곳에 저장 (한 번만 표시됨)

**중요**: API 키는 절대 공개하지 마세요!

### YouTube Transcript API 설정

#### 옵션 1: 직접 서버 구축 (권장)
```bash
# Python youtube-transcript-api 설치
pip install youtube-transcript-api flask

# 간단한 API 서버 실행
python api_server.py
```

#### 옵션 2: RapidAPI 사용
1. [RapidAPI](https://rapidapi.com) 가입
2. "YouTube Transcript" API 검색
3. 무료 플랜 구독
4. API 키 복사

---

## Make.com 시나리오 설정

### 1단계: Make.com 계정 생성

1. [Make.com](https://www.make.com) 접속
2. 무료 계정 생성
3. 대시보드로 이동

### 2단계: Blueprint Import

1. 좌측 메뉴에서 **Scenarios** 클릭
2. 우측 상단 **Create a new scenario** 클릭
3. 우측 하단 **...** (더보기) → **Import Blueprint** 선택
4. `make-scenarios/blueprint.json` 파일 업로드

### 3단계: 모듈별 설정

#### 모듈 1: Google Sheets - Search Rows
```
Spreadsheet ID: [복사한 Spreadsheet ID]
Sheet Name: 대기중_영상
Filter: status = '대기중'
```

#### 모듈 2: HTTP - Make a Request
```
URL: [YouTube Transcript API URL]
Method: POST
Headers:
  - Content-Type: application/json
  - Authorization: Bearer [YOUR_API_KEY]
Body: {"video_url": "{{1.youtube_url}}"}
```

#### 모듈 3: OpenAI - Create Completion (와비사비 각색)
```
Connection: [OpenAI API 연결]
Model: gpt-4
System Message: [wabisabi_script_prompt.txt 내용 붙여넣기]
User Message: {{2.data.transcript}}
Temperature: 0.7
Max Tokens: 3000
```

#### 모듈 4: OpenAI - Create Completion (SEO 생성)
```
Connection: [OpenAI API 연결]
Model: gpt-4
System Message: [seo_generation_prompt.txt 내용 붙여넣기]
User Message: {{3.choices[0].message.content}}
Temperature: 0.5
Max Tokens: 1500
```

#### 모듈 5: JSON - Parse JSON
```
JSON string: {{4.choices[0].message.content}}
```

#### 모듈 6: Google Sheets - Update a Row
```
Spreadsheet ID: [복사한 Spreadsheet ID]
Sheet Name: 완성_대본
Row Number: {{1.row_id}}
Values: [각 필드를 적절한 변수로 매핑]
```

#### 모듈 7: Telegram - Send a Message (선택)
```
Chat ID: [Telegram Chat ID]
Message: ✅ 새로운 와비사비 드라마 완성!
         제목: {{5.title}}
```

### 4단계: 시나리오 저장 및 활성화

1. 우측 상단 **Save** 클릭
2. 시나리오 이름: `YouTube 와비사비 드라마 자동화`
3. **Scheduling** 설정 (예: 매 1시간마다 실행)
4. **ON** 토글로 시나리오 활성화

---

## 프롬프트 파일 업로드

Make.com 모듈에서 프롬프트를 직접 복사-붙여넣기:

1. `prompts/wabisabi_script_prompt.txt` 파일 열기
2. 전체 내용 복사
3. Make.com 모듈 3 (OpenAI - 와비사비 각색)의 System Message에 붙여넣기
4. `{{RAW_TRANSCRIPT}}` 부분을 `{{2.data.transcript}}`로 수정
5. `prompts/seo_generation_prompt.txt`도 동일하게 모듈 4에 적용

---

## 테스트 실행

### 1단계: 테스트 데이터 입력

Google Sheets의 `대기중_영상` 시트에 테스트 URL 입력:

```
row_id: 1
youtube_url: https://www.youtube.com/watch?v=example123
status: 대기중
added_at: 2025-11-23 15:00:00
notes: 첫 테스트
```

### 2단계: 수동 실행

1. Make.com에서 시나리오 열기
2. 하단 **Run once** 클릭
3. 각 모듈의 실행 결과 확인

### 3단계: 결과 확인

- `완성_대본` 시트에 새 행이 추가되었는지 확인
- 와비사비 각색 대본 품질 확인
- SEO 메타데이터 검토

---

## 문제 해결

### 문제 1: YouTube Transcript API 오류
**증상**: "No transcript available"

**해결**:
1. 영상에 자막이 있는지 확인
2. 영상이 공개 상태인지 확인
3. API 키가 유효한지 확인

### 문제 2: OpenAI API Rate Limit
**증상**: "Rate limit exceeded"

**해결**:
1. Make.com에서 **Sleep** 모듈 추가 (30초)
2. API 플랜 업그레이드 고려
3. 요청 빈도 조절

### 문제 3: Google Sheets 권한 오류
**증상**: "Permission denied"

**해결**:
1. Make.com에서 Google Sheets 연결 재인증
2. 스프레드시트 공유 설정 확인
3. 편집 권한이 있는지 확인

### 문제 4: JSON 파싱 오류
**증상**: "Invalid JSON"

**해결**:
1. LLM 응답이 JSON 형식인지 확인
2. 프롬프트에 "JSON 형식으로만 출력" 명시 추가
3. JSON validator로 응답 검증

---

## 다음 단계

설정이 완료되었다면:

1. [자동화 플로우 문서](./automation_flow.md) 읽기
2. 프롬프트 커스터마이징
3. 배치 처리 설정
4. 비용 모니터링 설정

---

## 지원

문제가 발생하면:
- 프로젝트 README 참고
- Make.com 공식 문서 확인
- API 제공업체 지원 페이지 방문

**제작**: Claude Code 자동 생성
**업데이트**: 2025-11-23
