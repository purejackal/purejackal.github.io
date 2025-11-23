# Make.com 시나리오 가이드

이 디렉토리는 YouTube → 와비사비 드라마 자동화를 위한 Make.com 시나리오 파일을 포함합니다.

## 📁 파일 구성

- **blueprint.json**: Make.com에서 import할 수 있는 시나리오 템플릿

---

## 🎯 시나리오 개요

**이름**: YouTube 와비사비 드라마 자동화

**목적**: YouTube 영상 → RAW 대본 → 와비사비 각색 → SEO 생성 → Google Sheets 저장

**실행 방식**:
- 스케줄 (예: 매 1시간)
- 수동 실행 (Run once)

---

## 🧩 모듈 구성

### 1. Google Sheets - Search Rows
- **역할**: 대기 중인 YouTube URL 검색
- **필터**: `status = '대기중'`
- **출력**: row_id, youtube_url, status, added_at, notes

### 2. HTTP - Make a Request
- **역할**: YouTube Transcript API 호출
- **입력**: {{1.youtube_url}}
- **출력**: raw transcript (텍스트)

### 3. OpenAI - Create Completion (와비사비 각색)
- **역할**: RAW 대본을 와비사비 드라마로 각색
- **프롬프트**: wabisabi_script_prompt.txt
- **입력**: {{2.data.transcript}}
- **출력**: 각색된 대본 (커스텀 태그 포함)

### 4. OpenAI - Create Completion (SEO 생성)
- **역할**: SEO 메타데이터 생성
- **프롬프트**: seo_generation_prompt.txt
- **입력**: {{3.choices[0].message.content}}
- **출력**: JSON 형식 SEO 데이터

### 5. JSON - Parse JSON
- **역할**: SEO JSON 파싱
- **입력**: {{4.choices[0].message.content}}
- **출력**: title, subtitle, tags, keywords 등

### 6. Google Sheets - Update a Row
- **역할**: 완성 대본을 Google Sheets에 저장
- **시트**: 완성_대본
- **데이터**: 모든 단계의 출력 통합

### 7. Telegram - Send a Message (선택)
- **역할**: 완료 알림
- **메시지**: 제목, 카테고리, 상태 요약

---

## 📥 Import 방법

### 1단계: Make.com 접속
1. [Make.com](https://www.make.com) 로그인
2. 좌측 메뉴 **Scenarios** 클릭

### 2단계: Blueprint Import
1. **Create a new scenario** 클릭
2. 우측 하단 **...** (더보기) 메뉴
3. **Import Blueprint** 선택
4. `blueprint.json` 파일 업로드

### 3단계: 설정 커스터마이징
각 모듈을 클릭하여 다음 항목 수정:

#### 필수 수정 항목
- **Google Sheets 모듈**:
  - `spreadsheetId`: 본인의 Spreadsheet ID로 교체

- **HTTP 모듈**:
  - `url`: 실제 YouTube Transcript API URL
  - `Authorization`: 실제 API 키로 교체

- **OpenAI 모듈**:
  - Connection: OpenAI 계정 연결
  - System Message: 프롬프트 파일 내용 붙여넣기

- **Telegram 모듈** (선택):
  - `chatId`: 본인의 Telegram Chat ID

### 4단계: 테스트 실행
1. 하단 **Run once** 클릭
2. 각 모듈의 출력 확인
3. 에러 발생 시 설정 재확인

### 5단계: 스케줄 설정
1. 시계 아이콘 클릭
2. 실행 주기 설정 (예: Every 1 hour)
3. **ON** 토글로 활성화

---

## ⚙️ 커스터마이징 가이드

### 프롬프트 수정
`prompts/` 디렉토리의 프롬프트 파일을 수정한 후:
1. 수정된 내용 복사
2. Make.com 해당 모듈의 System Message 업데이트
3. 테스트 실행으로 검증

### 출력 필드 추가
Google Sheets에 새로운 열 추가 시:
1. `schemas/google_sheets_schema.json` 수정
2. Google Sheets에 새 열 추가
3. Make.com 모듈 6의 매핑에 새 필드 추가

### LLM 모델 변경
GPT-4 → GPT-3.5-turbo 변경 시:
1. 모듈 3, 4의 `model` 파라미터 수정
2. `max_tokens` 조정 (GPT-3.5는 최대 4096)
3. 비용 및 품질 모니터링

---

## 🔍 디버깅

### 문제: 모듈이 실행되지 않음
**확인사항**:
- 이전 모듈의 출력이 정상인지 확인
- 필터 조건이 올바른지 확인
- API 키가 유효한지 확인

### 문제: JSON 파싱 오류
**해결**:
1. LLM 응답 확인 (모듈 4 출력)
2. 응답이 JSON 형식이 아니면 프롬프트 수정
3. "JSON 형식으로만 출력하세요" 강조

### 문제: Google Sheets 업데이트 실패
**해결**:
1. Spreadsheet ID 확인
2. 시트 이름 정확성 확인
3. Make.com의 Google Sheets 연결 재인증

---

## 📊 시나리오 메트릭

### 실행 통계 확인
1. 시나리오 대시보드에서 **History** 탭 클릭
2. 성공/실패 비율 확인
3. 평균 실행 시간 확인

### 비용 모니터링
1. OpenAI Dashboard에서 API 사용량 확인
2. Make.com에서 Operations 사용량 확인
3. 월간 비용 추정

---

## 🚀 고급 기능

### 조건부 분기
특정 조건에 따라 다른 프롬프트 사용:
1. **Router** 모듈 추가
2. 조건 설정 (예: 영상 길이, 카테고리)
3. 각 경로에 다른 LLM 모듈 연결

### 에러 핸들러
에러 발생 시 자동 처리:
1. 모듈 우클릭 → **Add error handler**
2. 에러 로그를 Google Sheets에 기록
3. Telegram으로 에러 알림

### 병렬 처리
여러 URL 동시 처리:
1. **Array aggregator** 모듈 추가
2. 병렬 처리 설정
3. Pro 플랜 필요

---

## 📦 백업 및 버전 관리

### Blueprint 백업
1. 시나리오 우클릭 → **Export blueprint**
2. JSON 파일 저장
3. Git에 커밋

### 버전 히스토리
Make.com은 자동 버전 관리 제공:
1. 시나리오 **...** 메뉴
2. **Revisions** 선택
3. 이전 버전으로 복원 가능

---

## 🔗 관련 문서

- [설정 가이드](../docs/setup_guide.md)
- [자동화 플로우](../docs/automation_flow.md)
- [프로젝트 README](../README.md)

---

**제작**: Claude Code 자동 생성
**업데이트**: 2025-11-23
