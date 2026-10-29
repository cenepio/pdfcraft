# Codex 작업 지시서 v2 — PdfCraft 전체 한국어화 및 한글 PDF 편집 안정화

**대상 저장소:** https://github.com/cenepio/pdfcraft  
**검토 PR:** https://github.com/cenepio/pdfcraft/pull/1 (Draft)  
**작업 브랜치:** \`feat/korean-pdf-edit-v015\`  
**기준:** upstream PdfCraft v0.15.0 기반 Fork. 실제 현재 커밋·상태를 먼저 조회하고 진행.

## 0. 역할·목표·중요 제한

당신은 Rust 기반 PDF 편집 엔진, Unicode/IME, 폰트 임베딩 및 PDF 보존성 전문 개발자다. 기존 PR #1을 **완성품으로 간주하지 말고 개발 중인 변경안으로 검증**하라. 최종 목표는 한국어 Windows 사용자에게 알PDF보다 실용적이고 안정적인 한글 PDF 편집 경험을 제공하는 것이다. 실제 비교시험 없이 "알PDF 능가"라고 주장하지 않는다.

- 코드/테스트/검증/문서까지 실제 수행하라. 계획만 제출하거나 테스트를 통과한 것으로 꾸미지 말라.
- \`main\`에 직접 commit/push/merge하지 말고 **기존 작업 브랜치**를 계속 사용하라. PR #1을 업데이트한다. 새 저장소·중복 PR은 만들지 않는다.
- \`PowerMedic\`, \`PythonWorkspace\` 및 무관한 저장소는 수정하지 않는다. 다른 사람의 변경과 작업트리를 덮어쓰지 않는다.
- \`AGENTS.md\`가 최우선이며 \`CLAUDE.md\`도 준수한다. Rust 제품·빌드 규칙, 계층 분리, 무단 폰트/이미지 금지, panic/unsafe 금지, 원본 보존, CLI/automation 접근성 원칙을 따른다.
- PDF 기초 객체를 무단으로 재작성하거나 원본의 서명·권한·메타데이터·첨부파일을 조용히 제거하지 않는다.
- 실제 고객의 보험증권/개인정보를 공개 GitHub에 절대 올리지 않는다. 합성 데이터로 테스트한다.

## 1. 시작 전 현황 파악

1. 다음을 확인하고 정확한 HEAD, 빌드 환경, Git 작업트리 상태를 기록한다.

   \`\`\`bash
   git status --short
   git branch --show-current
   git log -1 --oneline
   git diff main...HEAD --stat
   gh pr view 1 --repo cenepio/pdfcraft --json number,state,isDraft,headRefName,baseRefName,url
   \`\`\`

2. \`AGENTS.md\`, \`CLAUDE.md\`, \`crates/fonts/README.md\`, \`docs/localization.md\`, \`docs/ko/README.md\`, \`docs/ko/ALPDF_COMPARISON.md\` 및 PR #1 diff를 읽는다.
3. \`crates/edit/src/text/fallback.rs\`, \`crates/edit/src/text/rewrite.rs\`, \`crates/edit/src/text/blocks.rs\`, \`crates/edit/src/text/editor.rs\`, \`crates/fonts/src/craft.rs\`, \`crates/fonts/src/embedding.rs\`, \`crates/fonts/src/pdf.rs\`, \`crates/ui-egui/src/theme.rs\`, \`crates/ui-egui/src/i18n/ko.tsv\`, 그리고 Windows IME 연동 코드를 추적한다.
4. 이미 업스트림에 고쳐진 부분, 열려 있는 관련 issue/PR, 기존 테스트를 먼저 검색한다. 기존 코드를 전면 재작성하지 않는다.
5. 다음 5개 범주로 결함 목록을 만든다: **P0 문서 손상**, **P1 한글 입력/저장 오류**, **P1 글꼴·문자 매핑 오류**, **P2 한글 UI 누락**, **P2 사용성/성능**. 재현 가능한 것부터 수정한다.

## 2. 1순위: 현재 PR #1의 잠재적 오류 수정

특히 다음 문제는 **가설로 시작하여 코드·시험으로 확인하고**, 확인된 결함만 정확하게 수정하라.

### 2.1 폴백 폰트 혼합의 정확성

- 현재 \`type3_faces()\`/ \`type3_font()\`가 한글을 발견하면 Korean 후보를 먼저 사용하고, 다른 문자는 Japanese/Inter 등의 후보에서 글리프를 찾는다.
- 서로 다른 원본 글꼴에서 가져온 문자들을 **단일 Type 3 폰트로 작성할 때** 실제 글리프 경로, advance width, 기준선, \`FontDescriptor\`, \`/Name\`, \`ToUnicode\`, \`ActualText\`의 일치 여부를 검사한다.
- 한글+영문+숫자, 한글+한자, 초성·중성·종성 자모, 기호, 공백, 줄바꿈, 화폐 단위, 굵은 글씨, serif/sans, 기존 서브셋 폰트의 모든 조합에서 **바른 글꼴·실제 글리프·UTF-8 논리 텍스트**가 유지되어야 한다.
- 필요하다면 **글리프가 실제로 유래한 폰트별로 Type 3 자원을 분리하고 텍스트 run을 생성**한다. 폰트 이름만 Korean으로 표시해 개선된 것처럼 보이게 하지 말라.
- 폴백 결정은 실제 \`cmap\`/glyph coverage 및 style을 우선하며, \`Hang\`/\`Kore\` 메타 태그만으로 실제 모든 한글·한자 글리프가 있다고 가정하지 않는다.
- \`Korean\` 폰트가 없는 빌드에서 일본어 폰트의 우연한 글리프로 잘못된 결과를 그리거나 조용히 '?'를 삽입하지 않는다. 충분한 글리프가 없다면 **문서를 변경하기 전에 오류를 반환**하고 적법한 글꼴 선택 방법을 한국어로 설명한다.

### 2.2 PDF 원본 편집·저장 정확성

- \`source_encodes()\`로 원본 폰트에서 인코딩 가능한 문자는 기존 폰트와 스타일을 재사용하고, 필요할 때만 라이선스에 맞는 명시적 폴백/임베딩으로 전환한다.
- 원문 일부만 변경했을 때 바뀌지 않은 텍스트 런의 글꼴·위치·색상·크기·자간·행간·회전, 표 선, 이미지 및 레이어를 최대한 유지한다.
- \`ToUnicode\`가 손상/누락된 문서, Type0/CID/CMap, 내장 부분집합, 합자, 두 글자 이상을 한 글리프에 매핑한 경우는 특성별 지원/제한을 명확히 판정한다.
- 저장→재열기→텍스트 재추출→렌더링 비교를 필수로 수행한다. 수정하지 않은 문서는 저장 후 변형되면 안 된다.
- overflow, 글자 겹침, 폰트 누락, 객체 손상 가능성이 있으면 사용자에게 경고 또는 명확한 오류를 주고 원본을 보존한다.
- 서명 문서를 편집하면 전자서명이 무효화될 수 있음을 사전에 명확히 경고하고, 임의로 서명을 보존했다고 주장하지 않는다. 스캔 PDF는 OCR 기반 편집과 텍스트 오버레이를 구별해 안내한다.

### 2.3 한국어 IME와 UI 폰트

- Windows 11 Microsoft Korean IME의 조합 시작/변경/확정/취소, Backspace/Delete, 방향키, 포커스 이동, Ctrl+C/V/Z/Y, 여러 줄 편집, 한국어+영문 전환을 실제 창에서 점검한다.
- 조합 중 글자가 두 번 들어가거나, 미완성 글자가 저장되거나, 입력 포커스 변경 시 텍스트가 유실되는 문제를 회귀시험으로 잡는다.
- \`theme.rs\`의 Korean 폰트 우선순위를 검증한다. 기본 글꼴/installed fallback, \`CRAFT_FONTS_DIR\` 유무, Windows 시스템 글꼴 유무에서 네모 글리프(□), 혼합 baseline/clip을 확인한다.
- UI 표시용 OS 설치 글꼴을 PDF에 자동으로 복사/임베딩하지 않는다.

## 3. 전체 한국어 UI 완성도

- \`crates/ui-egui/src/i18n/ko.tsv\`의 현재 번역과 PR #1의 **누락 17개 추가 항목**부터 검증한다.
- \`@id\`, \`@plural\`, \`key\` 컨텍스트, placeholders (\`{}\`, \`{n}\`, \`{fields}\` 등), 줄임표 \`…\`, 탭 및 escape 형식, 중복 키를 검사한다.
- \`cargo xtask i18n-report\`와 실제 UI 문자열 목록을 교차 검사한다. **정적 카탈로그 충족률과 실행 화면의 한글화율을 구별**한다.
- 메뉴, 도구모음, 파일/편집/보기/페이지, 환경설정, PDF 텍스트 편집, 양식, 주석, 인쇄, 서명, 저장, 도움말, 단축키 명칭, 확인/취소 창, 경고·오류 메시지까지 한국어로 자연스럽게 표현한다.
- 내부 명령 ID, 자동화 API, CLI 출력 형식을 무작정 번역해 호환성을 깨지 않는다. 시스템/엔진 오류 원문은 진단용으로 유지하면서 사용자에게 읽기 쉬운 한국어 설명을 제공한다.
- 번역의 과도한 길이로 버튼·패널 텍스트가 잘리거나 입력칸이 사라지지 않도록 100/125/150/200% UI 배율에서 확인한다.
- 개발자 및 제품 원저작자 저작권 표시, 라이선스·프로그램명 등 변경하면 안 되는 문자열은 유지한다.

## 4. 테스트 설계 및 객관적 합격 기준

**먼저 실패하는 재현 테스트를 만들고 수정 후 통과시켜라.** 다음 synthetic fixture들을 생성하되, 개인정보/유료 소프트웨어 리소스는 포함하지 않는다.

- A. 순수 한글: \`보험가입금액\` → \`보험가입금액 변경\`
- B. 숫자/금액: \`1,000,000,000원\` → \`1,500,000,000원\`
- C. 한글+영문+숫자: \`보험료 Premium 2026 ₩1,250,000\`
- D. 다국어: \`한국어 漢字 English 123\`
- E. 자모/특수문자: \`가 / ㄱㅏ / 가 / ㎡ / ㈜ / ①\`
- F. 표(셀 내부의 숫자·주소·특약명), 다중 열, 장문 줄바꿈, 굵기/크기 혼합
- G. Type0/CID, 정상·손상·없는 ToUnicode, 부분 내장 폰트, 글리프 없음
- H. 서명·암호/권한 제한 문서, 스캔 이미지 PDF, 빈 페이지, 손상 PDF

각 시험은 최소 다음을 검증한다.

1. 실행 성공 또는 **설명 가능한 안전한 실패** (앱 크래시 없음).
2. 성공 시 저장한 파일의 reopen 및 수정 텍스트의 Unicode 재추출이 기대값과 일치.
3. 편집하지 않은 객체/텍스트/표/이미지 위치가 허용 오차 안에서 보존되었는지 검증. 허용 오차는 시험 목적에 맞게 미리 정의하고 결과와 함께 제시.
4. 오버플로·폰트 누락·불가능한 인코딩 시 손상된 PDF를 출력하지 않고 원본 유지.
5. 한글/중국어/일본어/영어 기존 편집 테스트의 회귀 없음.

알PDF와 동일한 Windows 장치·동일 문서·동일 변경 조건에서 **수정 성공률, Unicode 정확도, 주변 서식 보존, 저장·재열기 성공률, 작업 단계 수, 처리시간**을 측정하고 결과표를 만든다. 알PDF를 자동화하거나 실측할 환경이 없다면 **"알PDF 직접 비교 미실시"**로 기록하고 승패 결론을 쓰지 않는다.

## 5. 실행해야 할 품질 검증

작업 전/후에 결과를 기록하라. 설치된 의존성/환경이 허용되는 경우 다음을 우선 실행한다.

\`\`\`bash
cargo fmt --all -- --check
cargo check -p pdfcraft-fonts -p pdfcraft-edit -p pdfcraft-ui-egui -j 2
cargo test -p pdfcraft-fonts -j 2
cargo test -p pdfcraft-edit -j 2
cargo test -p pdfcraft-ui-egui -j 2
cargo clippy -p pdfcraft-fonts -p pdfcraft-edit -p pdfcraft-ui-egui --all-targets -- -D warnings
cargo xtask i18n-report
cargo xtask assets
cargo xtask layers
\`\`\`

- 별도 \`storytold/craft-fonts\`의 원저작물·라이선스와 **현재 저장소가 고정한 버전**을 확인해 \`CRAFT_FONTS_DIR\` **설정/미설정** 모두 검증한다.
- 브랜치에서 가능한 Windows UI/문서 실기시험을 시행한다. 환경이 Linux-only/헤드리스이면 미수행 이유를 기록하고 Windows 테스트를 통과했다고 표기하지 않는다.
- 필요 시 GUI는 공식 \`--control\`/CLI UI 명령, 엔진은 \`pdfcraft-cli run --script\` 및 \`page_render\`로 시험한다. UI control 및 MCP는 opt-in-only 정책 유지.
- 전체 \`cargo xtask ci\`는 환경과 리소스가 허용될 때 시행한다. 실패한 테스트를 삭제하거나 기능을 비활성화해서 통과 처리하지 않는다.

## 6. 커밋·문서·최종 보고

1. 결함 수정과 테스트는 의미 있는 작은 commit으로 PR #1의 기존 브랜치에 push한다. 사용자 요청 전 \`main\` 병합 금지.
2. \`docs/ko/README.md\`, \`docs/ko/ALPDF_COMPARISON.md\` 및 본 지시서의 진행/미검증 항목을 실측 근거로 업데이트한다.
3. 리포트 \`docs/ko/VALIDATION_REPORT.md\`를 작성한다. 필수 항목:
   - 기준 commit, 환경(OS/Rust/폰트), 수정 파일과 변경 이유
   - 발견한 버그/재현 단계/수정 내용
   - 테스트 명령·성공·실패·미실행과 로그 또는 증거 경로
   - 한글 추출/렌더링/편집 회귀시험 결과표, 미지원 조건
   - 알PDF 직접 비교 결과 (미실시 시 정확히 미실시 표기)
   - 남은 결함, 위험도, 재현 방법, 다음 우선순위
4. PR 설명을 새 상태로 업데이트하되 실제 테스트가 끝나기 전에는 Draft를 유지한다.

**종료 조건:** 검증 가능한 코드·테스트·리포트를 PR #1에 반영하고, 수행한 것과 수행하지 못한 것을 분리하여 한국어로 요약 보고한다. 단순 지시서 재작성/검토만으로 완료했다고 보고하지 않는다.
