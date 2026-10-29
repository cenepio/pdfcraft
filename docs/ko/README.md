# PdfCraft 한국어 UI 및 PDF 편집 개선 — 개발 브랜치

기준: `cenepio/pdfcraft` fork (upstream v0.15.0).

## 실제 코드 변경

- 한국어 UI 카탈로그의 누락된 17개 항목을 추가하였습니다.
- 별도 OFL 라이선스 `craft-fonts`의 `Hang`/`Kore` 글꼴을 한글 PDF 텍스트 교체 시 우선 사용합니다.
- 한국어 UI에서 한글 글꼴을 일본어 CJK 글꼴보다 먼저 사용합니다.
- 한국어 폴백 글꼴 자원을 `PCKorean`/`PCKo`로 식별합니다.
- 한글·영문·숫자 혼합 시 글리프 대체 후보를 탐색합니다.
- 관련 회귀 테스트를 추가합니다.

## 주의

이 브랜치는 **개발 패치**입니다. Windows 11 실기시험, 전체 Rust 빌드 및 알PDF 비교시험을 통과했다고 주장할 수 없습니다. 메인 브랜치와 릴리스는 검증 전까지 유지해야 합니다. 변경 후 재열기, 글꼴 유지 및 문서 시각적 충실도를 반드시 검사해야 합니다.

## 검증 명령

```bash
cargo fmt --all -- --check
cargo test -p pdfcraft-fonts
cargo test -p pdfcraft-edit
cargo test -p pdfcraft-ui-egui
cargo xtask i18n-report
```

선택적 별도 `storytold/craft-fonts` 체크아웃으로 `CRAFT_FONTS_DIR`을 설정한 경우도 확인하십시오. 별도 권한 없이 OS 설치 폰트를 PDF나 저장소에 자동 복사/포함하지 않습니다. 원본 `AGENTS.md`의 라이선스·상표·자산 규칙을 따릅니다.

## 초기 개발 패키지

이 작업의 원래 준비물은 별도 ZIP 패키지에 `scripts/apply_ko_patch.py`, `scripts/enhance_korean_editor.py`, `scripts/verify_ko_patch.py` 및 비교시험 명세가 포함됩니다. 이 브랜치는 해당 스크립트를 실행한 산출물과 같은 목표의 **직접 적용된 Rust 소스 변경**을 담고 있습니다.
