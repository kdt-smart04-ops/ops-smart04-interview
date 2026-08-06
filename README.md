# 스마트플밍 4기 — 모의면접 서류 준비 현황 대시보드

Google Drive의 학습자별 취업서류 폴더를 조회해 GitHub Pages에서 확인하는 정적 대시보드입니다.
운영진 확인용이며, 학습자에게 배포하는 페이지가 아닙니다.

`kim-waffle/smart4_check_documents`(기획반)를 기반으로 플밍반 드라이브에 맞춰 재구성했습니다.

---

## 1. 개인정보 취급

> 이 사이트는 URL을 아는 사람 누구나 접근할 수 있습니다. 접근 통제가 아니라 **노출 축소**만 적용돼 있습니다.

- 학습자 실명이 표시됩니다. 마스킹은 동명이인 충돌로 적용하지 않았습니다 (48명 중 6명 충돌)
- `noindex` 메타 태그와 `robots.txt`로 검색엔진 색인을 차단합니다
- Drive 링크는 노출되지만 **Drive 권한이 실제 접근을 막습니다**
- 조회 권한은 `drive.metadata.readonly` — **파일 내용은 읽지 않습니다.** 파일명·수정일·폴더 구조만 봅니다

**정리 시점: 2026-09-30 이후 저장소를 private으로 전환하거나 삭제합니다.**
private 전환 시 Pages 사이트도 함께 내려갑니다.

---

## 2. 데이터 출처

Drive 상위 폴더: `[스마트플밍 4기] 취업관리`

| 용도 | 폴더 | config 키 |
|---|---|---|
| 서류 | `훈련생 구직활동 자료` | `sourceFolderId` |
| 면접 | `모의면접` | `interviewSourceFolderId` |

기대하는 폴더 구조:

```
훈련생 구직활동 자료/          모의면접/
└── {학습자 이름}/            └── {팀}_모의면접 ({담당자})/
    ├── 1_입사지원서(...)/         └── {번호}. {학습자 이름}/
    ├── 2_포트폴리오/
    └── 3_면접자료/
```

- 하위 폴더는 **부분 문자열 매칭**이라 `1_`·`2_` 접두사와 괄호 부연이 붙어 있어도 인식됩니다
- 면접 폴더는 `{번호}. {이름}` 형식에서 이름을 추출해 서류 폴더 명단과 대조합니다
- 이름이 안 맞으면 `data/config.json`의 `interviewNameAliases`에 수동 매핑을 추가합니다
- 면접 폴더가 없는 학습자는 `not_applicable`로 표시됩니다

---

## 3. 구성

| 경로 | 역할 |
|---|---|
| `index.html` · `styles.css` · `app.js` | Pages로 배포되는 대시보드 |
| `data/config.json` | 폴더 ID와 문서 유형 설정 |
| `data/manual-status.json` | 조기취업 학습자 ID 목록 |
| `data/dashboard-data.json` | 현재 데이터 (Actions가 생성) |
| `data/history/` | 시간별 스냅샷 |
| `scripts/fetch_drive_status.py` | Drive 조회 및 JSON 생성 |
| `.github/workflows/` | 매시 25분 자동 갱신 + Pages 배포 |

---

## 4. GitHub Secrets

`Settings` → `Secrets and variables` → `Actions`에 등록합니다.

- `GOOGLE_CLIENT_ID`
- `GOOGLE_CLIENT_SECRET`
- `GOOGLE_REFRESH_TOKEN`
- `GOOGLE_DRIVE_ROOT_FOLDER_ID`

`GOOGLE_DRIVE_INTERVIEW_ROOT_FOLDER_ID`는 선택입니다. 미설정 시 `data/config.json` 값을 씁니다.

> OAuth 동의 화면을 **`테스트` 상태로 두면 refresh token이 7일 뒤 만료됩니다.**
> 반드시 `프로덕션`으로 게시하세요. "확인되지 않은 앱" 경고는 그대로 진행해도 됩니다.

---

## 5. 로컬 실행

```bash
python -m http.server 8000
```

데이터 갱신:

```bash
pip install -r requirements.txt
python scripts/fetch_drive_status.py
```

토큰 발급 (`credentials.json`을 저장소 루트에 둔 뒤):

```bash
python scripts/generate_oauth_token.py
```

`credentials.json`과 `token.json`은 `.gitignore`에 있어 커밋되지 않습니다.
