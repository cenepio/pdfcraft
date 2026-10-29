# Codex 작업 지시서 — PdfCraft 한국어 안정화

저장소: `cenepio/pdfcraft`, 브랜치: `feat/korean-pdf-edit-v015`. 사용자 승인 없이 `main`에 병합하지 마세요.

1. `cargo fmt --all -- --check`를 수행하고 포맷 및 컴파일 오류를 수정합니다.
2. `cargo check`와 `cargo test -p pdfcraft-fonts -p pdfcraft-edit -p pdfcraft-ui-egui`를 실행하고 실패 결과를 기록합니다.
3. `cargo xtask i18n-report`로 한국어 누락 및 매개변수/복수형/중복 검증을 수행합니다.
4. `CRAFT_FONTS_DIR`이 설정된 경우와 없는 경우를 시험하고, 권한 없는 시스템 폰트 임베딩을 금지합니다.
5. Windows 11에서 한국어 IME 조합/Backspace/붙여넣기, 한글 UI 네모 글리프 여부, 100/150/200% 스케일을 검사합니다.
6. 합성 보험증권·가입설계서 PDF에서 한글, 숫자, 통화 단위 수정 후 저장·재열기·유니코드 재추출 및 이미지 비교를 수행합니다.
7. CID/Type0, 부분 내장 글꼴, ToUnicode 누락, 서명 문서, 스캔 PDF는 기능별 미지원 사례와 실패 시 원본 보존 여부를 검사합니다.
8. 기존 영어·일본어·중국어 PDF 편집 회귀시험을 수행하고 부작용을 수정합니다.
9. 공인되지 않은 폰트/이미지/브랜드 자산을 저장소에 추가하지 않습니다. `AGENTS.md`와 `CLAUDE.md`의 더 엄격한 규칙을 유지합니다.

**정적 패치 작성과 Windows 실행·성능 검증은 다릅니다.** Rust 빌드와 동일 조건 알PDF 비교가 끝나기 전에는 '완료' 또는 '알PDF 능가'로 표기하지 않습니다.
