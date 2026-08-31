# Claude Code 완전 매뉴얼

- **문서 버전**: v1
- **작성일**: 2026년 8월 24일
- **작성 목적**: Claude Code의 개요부터 고급 활용까지 언제든 참고할 수 있도록 체계적으로 정리
- **근거 자료**: Anthropic 공식 문서(code.claude.com/docs) 2026년 8월 24일 기준

> 본 문서의 명령어·설정 키·경로는 공식 문서를 근거로 정리하였습니다. Claude Code는 업데이트 주기가 매우 빠른 제품이므로, 실제 적용 전에는 `claude --version` 및 공식 문서로 재확인하시기를 권장드립니다.

---

## 목차

1. [Claude Code 개요](#1-claude-code-개요)
2. [설치 및 초기 설정](#2-설치-및-초기-설정)
3. [기본 사용법](#3-기본-사용법)
4. [CLI 명령어 레퍼런스](#4-cli-명령어-레퍼런스)
5. [CLI 플래그 레퍼런스](#5-cli-플래그-레퍼런스)
6. [슬래시 명령어와 번들 스킬](#6-슬래시-명령어와-번들-스킬)
7. [메모리: CLAUDE.md와 자동 메모리](#7-메모리-claudemd와-자동-메모리)
8. [스킬(Skills) 만들기](#8-스킬skills-만들기)
9. [서브에이전트(Subagents)](#9-서브에이전트subagents)
10. [훅(Hooks)](#10-훅hooks)
11. [MCP 연동](#11-mcp-연동)
12. [플러그인](#12-플러그인)
13. [권한과 보안](#13-권한과-보안)
14. [설정 파일 체계](#14-설정-파일-체계)
15. [자동화·스케줄링·CI 활용](#15-자동화스케줄링ci-활용)
16. [실무 활용 워크플로](#16-실무-활용-워크플로)
17. [문제 해결(Troubleshooting)](#17-문제-해결troubleshooting)
18. [용어 정리 및 참고 링크](#18-용어-정리-및-참고-링크)

---

## 1. Claude Code 개요

### 1.1 정의

Claude Code는 Anthropic이 제공하는 **에이전틱 코딩 도구(agentic coding tool)** 입니다. 단순한 코드 자동완성 도구가 아니라, 코드베이스 전체를 읽고 파일을 편집하며 셸 명령을 실행하고 개발 도구와 연동하여 작업을 수행하는 AI 에이전트입니다.

주요 특징은 다음과 같습니다.

- 프로젝트 전체 맥락을 이해하고 여러 파일에 걸친 작업을 수행합니다.
- Git 연동을 통해 커밋, 브랜치 생성, PR 작성까지 처리합니다.
- MCP(Model Context Protocol)를 통해 외부 시스템(Jira, Slack, Google Drive, DB 등)과 연결됩니다.
- 훅·스킬·서브에이전트·플러그인으로 조직 표준에 맞게 확장할 수 있습니다.

### 1.2 사용 가능한 표면(Surface)

| 표면 | 설명 | 비고 |
|---|---|---|
| 터미널 CLI | 가장 기능이 완전한 형태 | 서드파티 프로바이더 지원 |
| VS Code 확장 | 인라인 diff, @멘션, 플랜 리뷰 | Cursor에도 설치 가능 |
| JetBrains 플러그인 | IntelliJ, PyCharm, WebStorm 등 | CLI 별도 설치 필요 |
| 데스크톱 앱 | 시각적 diff 리뷰, 다중 세션, 예약 작업 | macOS / Windows / Ubuntu·Debian(베타) |
| 웹 | claude.ai/code, 로컬 설치 불필요 | 모바일 앱(iOS/Android) 포함 |

모든 표면이 동일한 엔진을 사용하므로, 저장소의 `CLAUDE.md`, 설정, MCP 서버 구성이 표면 간에 공유됩니다.

### 1.3 대표 활용 사례

- 테스트 코드 작성 및 실패 테스트 수정
- 버그 원인 추적 및 수정
- 린트 오류 일괄 정리, 의존성 업데이트, 릴리스 노트 작성
- 커밋 메시지 작성 및 PR 생성
- CI/CD 파이프라인 내 자동 코드 리뷰 및 이슈 트리아지
- 로그 파이핑을 통한 이상 징후 분석

---

## 2. 설치 및 초기 설정

### 2.1 설치 방법

**네이티브 설치(권장)**

| 환경 | 명령어 |
|---|---|
| macOS / Linux / WSL | `curl -fsSL https://claude.ai/install.sh \| bash` |
| Windows PowerShell | `irm https://claude.ai/install.ps1 \| iex` |
| Windows CMD | `curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd` |

네이티브 설치본은 백그라운드에서 자동 업데이트됩니다.

**패키지 매니저 설치**

| 매니저 | 명령어 | 자동 업데이트 |
|---|---|---|
| Homebrew | `brew install --cask claude-code` | 미지원(`brew upgrade` 필요) |
| WinGet | `winget install Anthropic.ClaudeCode` | 미지원(`winget upgrade` 필요) |
| apt / dnf / apk | 공식 문서의 Linux 패키지 매니저 안내 참조 | [확인필요] |

> Homebrew는 두 가지 cask를 제공합니다. `claude-code`는 안정 채널(통상 약 1주 지연), `claude-code@latest`는 최신 채널을 추종합니다.

**Windows 사용자 참고사항**

네이티브 Windows 환경에서는 Bash 도구 사용을 위해 [Git for Windows](https://git-scm.com/downloads/win) 설치를 권장드립니다. 미설치 시 Claude Code는 PowerShell을 셸 도구로 사용합니다. WSL 환경에서는 별도 설치가 불필요합니다.

### 2.2 최초 실행

```bash
cd your-project
claude
```

최초 실행 시 로그인 절차가 진행됩니다. `ANTHROPIC_API_KEY` 환경변수가 설정되어 있는 경우 로그인 프롬프트를 건너뛰고 해당 키의 사용 승인만 요청합니다.

### 2.3 프로젝트 초기화

```
/init
```

Claude가 코드베이스를 분석하여 빌드 명령, 테스트 방법, 프로젝트 관례를 담은 `CLAUDE.md` 초안을 생성합니다. 기존 파일이 있는 경우 덮어쓰지 않고 개선안을 제안합니다.

`CLAUDE_CODE_NEW_INIT=1` 환경변수를 설정하면 대화형 다단계 플로우가 활성화되어, CLAUDE.md·스킬·훅 중 무엇을 설정할지 선택하고 서브에이전트가 코드베이스를 탐색한 뒤 검토 가능한 제안서를 제시합니다.

### 2.4 설치 진단

```bash
claude doctor        # 설치·설정 진단(읽기 전용)
claude --safe-mode   # 모든 커스터마이징 비활성화 상태로 실행
```

세션 내에서는 `/doctor` 스킬로 설정 점검 및 자동 수정을 수행할 수 있습니다.

---

## 3. 기본 사용법

### 3.1 대화형 세션

```bash
claude                          # 대화형 세션 시작
claude "이 프로젝트를 설명해줘"    # 초기 프롬프트와 함께 시작
claude -c                       # 현재 디렉터리의 최근 대화 이어가기
claude -r "auth-refactor" "이 PR 마무리해줘"   # 세션 ID/이름으로 재개
```

### 3.2 비대화형(헤드리스) 실행

```bash
claude -p "이 함수를 설명해줘"                  # 응답 출력 후 종료
cat logs.txt | claude -p "이상 징후를 알려줘"     # 파이프 입력 처리
git diff main --name-only | claude -p "변경 파일의 보안 이슈를 검토해줘"
```

`-p` 모드는 CI/CD, 배치 스크립트, 자동화 파이프라인에 적합합니다.

### 3.3 권장 작업 흐름

1. **탐색**: 먼저 코드베이스를 읽게 하여 맥락을 확보합니다.
2. **계획**: 큰 변경은 `/plan`(플랜 모드)으로 계획을 먼저 수립합니다.
3. **실행**: 계획 승인 후 실제 편집을 수행하게 합니다.
4. **검증**: `/verify`, 테스트 실행, `/diff`로 결과를 확인합니다.
5. **커밋**: 변경 내역을 커밋하고 필요 시 PR을 생성합니다.

### 3.4 컨텍스트 관리

| 명령 | 용도 |
|---|---|
| `/context` | 현재 컨텍스트 사용량을 시각화 |
| `/compact [지시사항]` | 대화를 요약하여 컨텍스트 확보 |
| `/clear` | 컨텍스트를 비우고 새 대화 시작 |
| `/autocompact [auto\|토큰수]` | 자동 압축 임계값 설정 |
| `/rewind [대상]` | 코드와 대화를 체크포인트로 롤백 |

---

## 4. CLI 명령어 레퍼런스

### 4.1 세션 실행

| 명령 | 설명 |
|---|---|
| `claude` | 대화형 세션 시작 |
| `claude "질의"` | 초기 프롬프트와 함께 시작 |
| `claude -p "질의"` | 비대화형 실행 후 종료 |
| `claude -c` | 현재 디렉터리 최근 대화 이어가기 |
| `claude -r "<세션>" "질의"` | 세션 ID 또는 이름으로 재개 |

### 4.2 설치·인증·진단

| 명령 | 설명 |
|---|---|
| `claude update` | 최신 버전으로 업데이트 |
| `claude install [버전]` | 네이티브 바이너리 설치/재설치 |
| `claude auth login` | Anthropic 계정 로그인 |
| `claude auth logout` | 로그아웃 |
| `claude auth status` | 인증 상태를 JSON으로 출력 |
| `claude doctor` | 설치·설정 진단(읽기 전용) |
| `claude setup-token` | CI·스크립트용 장기 OAuth 토큰 생성 |
| `claude import [codex\|gemini]` | 타 코딩 에이전트 설정 가져오기 |

### 4.3 백그라운드 세션 관리

| 명령 | 설명 |
|---|---|
| `claude agents` | 병렬 백그라운드 세션 모니터링·디스패치 뷰 |
| `claude attach <id>` | 백그라운드 세션에 현재 터미널로 접속 |
| `claude logs <id>` | 백그라운드 세션의 최근 출력 확인 |
| `claude stop <id>` | 백그라운드 세션 중지 |
| `claude respawn <id>` | 대화 맥락을 유지한 채 세션 재시작 |
| `claude rm <id>` | 목록에서 백그라운드 세션 제거 |
| `claude daemon status` | 백그라운드 세션 감독자 상태·진단 출력 |
| `claude daemon stop --any` | 감독자 및 하위 세션 중지 |

### 4.4 확장 기능 관리

| 명령 | 설명 |
|---|---|
| `claude mcp` | MCP 서버 구성 |
| `claude mcp login <name>` / `logout <name>` | MCP 서버 OAuth 인증/해제 |
| `claude plugin` | 플러그인 관리 |
| `claude remote-control` | Remote Control 서버 시작 |
| `claude gateway` | 자체 호스팅 게이트웨이 서버 시작 |
| `claude self-hosted-runner` | 자체 호스팅 환경 러너 프로세스 시작 |
| `claude project purge [경로]` | 해당 프로젝트의 로컬 상태 전체 삭제 |
| `claude ultrareview [대상]` | 비대화형 심층 리뷰 실행 |
| `claude auto-mode defaults` / `reset` | 자동 모드 분류기 규칙 출력/초기화 |

> `claude project purge`는 로컬 상태를 삭제하는 명령이므로, 실행 전 `--dry-run`으로 대상 확인을 권장드립니다.

---

## 5. CLI 플래그 레퍼런스

### 5.1 세션 관련

| 플래그 | 설명 |
|---|---|
| `--print`, `-p` | 비대화형 출력 |
| `--continue`, `-c` | 최근 대화 이어가기 |
| `--resume`, `-r` | 특정 세션 재개 |
| `--name`, `-n` | 세션 표시 이름 설정 |
| `--background`, `--bg` | 백그라운드 에이전트로 시작 |
| `--fork-session` | 원본 세션을 유지한 채 새 세션 ID 생성 |
| `--session-id` | 특정 UUID를 세션 ID로 사용 |
| `--no-session-persistence` | 세션 저장 비활성화 |
| `--bare` | 훅·스킬·명령 자동 탐색 생략(최소 모드) |

### 5.2 모델·구성

| 플래그 | 설명 |
|---|---|
| `--model` | 모델 지정(별칭 또는 전체 이름) |
| `--effort` | 사고 강도 지정(low, medium, high, xhigh, max, ultracode) |
| `--fallback-model` | 자동 폴백 모델 지정 |
| `--advisor <model>` | 서버 측 어드바이저 도구 활성화 |
| `--autocompact <auto\|토큰>` | 자동 압축 임계값 지정 |
| `--betas` | 베타 헤더 지정(API 키 사용자 전용) |

### 5.3 시스템 프롬프트

| 플래그 | 설명 |
|---|---|
| `--system-prompt` | 시스템 프롬프트 전체 교체 |
| `--system-prompt-file` | 파일에서 시스템 프롬프트 로드 |
| `--append-system-prompt` | 기본 시스템 프롬프트에 텍스트 추가 |
| `--append-system-prompt-file` | 파일 내용을 추가 |
| `--append-subagent-system-prompt` | 모든 서브에이전트에 공통 지시 추가 |

### 5.4 권한·보안

| 플래그 | 설명 |
|---|---|
| `--permission-mode` | 시작 권한 모드 지정 |
| `--allowedTools` | 승인 없이 실행 가능한 도구 지정 |
| `--disallowedTools` | 거부할 도구 지정 |
| `--dangerously-skip-permissions` | 권한 프롬프트 생략(격리 환경 전용) |
| `--permission-prompt-tool` | 권한 승인을 처리할 MCP 도구 지정 |

### 5.5 파일·디렉터리·확장

| 플래그 | 설명 |
|---|---|
| `--add-dir` | 추가 작업 디렉터리 지정 |
| `--plugin-dir` | 디렉터리 또는 .zip에서 플러그인 로드 |
| `--plugin-url` | URL에서 플러그인 .zip 가져오기 |
| `--mcp-config` | JSON 파일/문자열에서 MCP 서버 로드 |
| `--strict-mcp-config` | `--mcp-config`의 서버만 사용 |
| `--settings` | 설정 JSON 파일 또는 인라인 JSON 지정 |
| `--setting-sources` | 사용할 설정 소스 지정(user, project, local) |

### 5.6 출력·디버그

| 플래그 | 설명 |
|---|---|
| `--output-format` | 출력 형식(text, json, stream-json) |
| `--input-format` | 입력 형식(text, stream-json) |
| `--json-schema` | JSON Schema에 부합하는 검증된 출력 요청 |
| `--verbose` | 상세 출력 |
| `--include-partial-messages` | 부분 스트리밍 이벤트 포함 |
| `--include-hook-events` | 훅 생명주기 이벤트 포함 |
| `--debug[=카테고리]` | 디버그 모드(예: `--debug='mcp,startup'`) |
| `--debug-file <경로>` | 디버그 로그 파일 경로 지정 |
| `--safe-mode` | 커스터마이징 전면 비활성화 |

### 5.7 실행 제어·원격

| 플래그 | 설명 |
|---|---|
| `--max-turns` | 에이전트 턴 수 제한 |
| `--max-budget-usd` | API 호출 최대 비용 한도 |
| `--agent` | 메인 세션으로 사용할 에이전트 지정 |
| `--agents` | JSON으로 서브에이전트 동적 정의 |
| `--cloud` | claude.ai 웹 세션 생성/연결 |
| `--teleport` | 웹 세션을 로컬 터미널로 가져오기 |
| `--remote-control`, `--rc` | Remote Control 활성화 |
| `--environment <id>` | 자체 호스팅 환경에서 세션 생성 |
| `--ide` | 시작 시 IDE 자동 연결 |
| `--chrome` / `--no-chrome` | Chrome 브라우저 연동 활성화/비활성화 |
| `--exec` | 셸 명령을 PTY 백그라운드 작업으로 실행 |
| `--init` / `--init-only` / `--maintenance` | Setup 훅 관련 실행 제어 |

---

## 6. 슬래시 명령어와 번들 스킬

### 6.1 주요 내장 명령어

**세션·컨텍스트 관리**

| 명령 | 설명 |
|---|---|
| `/help` | 도움말 및 명령 목록 |
| `/clear [이름]` | 컨텍스트를 비우고 새 대화 시작 |
| `/compact [지시]` | 대화 요약으로 컨텍스트 확보 |
| `/context [all]` | 컨텍스트 사용량 시각화 |
| `/resume [이름]` | 이전 대화로 복귀 |
| `/rewind [대상]` | 코드·대화 롤백 |
| `/branch [이름]` | 대화 분기 생성 |
| `/recap` | 현재 세션 한 줄 요약 |
| `/export [파일명]` | 대화를 텍스트로 내보내기 |
| `/status` | 세션 상태 확인 |
| `/usage`, `/cost` | 토큰 사용량 및 비용 확인 |
| `/exit` | 종료 |

**작업 제어**

| 명령 | 설명 |
|---|---|
| `/plan [설명]` | 플랜 모드 진입 |
| `/goal [조건\|clear]` | 목표 설정 |
| `/subtask <지시>` | 서브에이전트에 부수 작업 위임 |
| `/background [프롬프트]` | 백그라운드 에이전트로 분리 |
| `/tasks` | 백그라운드 작업 목록 |
| `/btw [질문]` | 대화에 추가하지 않고 곁가지 질문 |
| `/diff` | 미커밋 변경사항 diff 뷰어 |
| `/verify` | 코드 정확성 검증 |
| `/simplify [강도]` | 가독성·유지보수성 개선 |
| `/security-review` | diff의 보안 취약점 점검 |

**설정·확장**

| 명령 | 설명 |
|---|---|
| `/config`, `/settings` | 설정 인터페이스 |
| `/permissions` | 허용/질문/거부 규칙 관리 |
| `/hooks` | 훅 구성 확인 |
| `/mcp [reconnect\|enable\|disable]` | MCP 서버 관리 |
| `/agents` | 서브에이전트 구성 관리 |
| `/plugin [하위명령]` | 플러그인 관리 |
| `/memory` | CLAUDE.md 편집 및 자동 메모리 관리 |
| `/model [모델]` | 모델 전환 및 기본값 저장 |
| `/add-dir <경로>` | 작업 디렉터리 추가 |
| `/cd <경로>` | 작업 디렉터리 이동 |
| `/ide` | IDE 연동 관리 |
| `/theme [테마]` | UI 테마 설정 |
| `/keybindings` | 단축키 설정 파일 열기 |

**표면 이동**

| 명령 | 설명 |
|---|---|
| `/desktop` | 데스크톱 앱에서 세션 계속 |
| `/teleport` | 웹 세션을 터미널로 가져오기 |
| `/mobile` | 모바일 앱 다운로드 QR 표시 |

### 6.2 번들 스킬

번들 스킬은 프롬프트 기반으로 동작하며, Claude가 도구를 조합해 작업을 수행합니다.

| 스킬 | 설명 |
|---|---|
| `/init` | 프로젝트 CLAUDE.md 초기 생성 |
| `/doctor` | 설정 점검 및 자동 수정 |
| `/code-review [강도] [--fix] [--comment] [대상]` | diff·PR 리뷰 및 수정/코멘트 |
| `/debug [설명]` | 디버그 로깅 활성화 및 문제 해결 |
| `/batch <지시>` | 코드베이스 전반 대규모 변경을 병렬 수행 |
| `/deep-research <질문>` | 웹 검색 확장 후 출처 포함 리포트 작성 |
| `/loop [주기] [프롬프트]` | 프롬프트 반복 실행 |
| `/insights` | 최근 세션·사용 패턴 HTML 리포트 |
| `/dataviz [요청]` | 차트·대시보드 설계 가이드 |
| `/claude-api [하위명령]` | Claude API 레퍼런스 로드 및 마이그레이션 |
| `/fewer-permission-prompts` | 허용목록 제안으로 권한 프롬프트 감소 |
| `/import [codex\|gemini]` | 타 에이전트 설정 가져오기 |
| `/install-github-app` | GitHub App 설치 |
| `/install-slack-app` | Slack 앱 설치 |
| `/run` | 앱을 실행해 변경사항 확인 |
| `/run-skill-generator` | `/run`·`/verify`용 프로젝트별 실행 레시피 기록 |

번들 스킬 전체를 비활성화하려면 `disableBundledSkills` 설정을 사용합니다(단 `/doctor`는 예외적으로 유지됩니다).

---

## 7. 메모리: CLAUDE.md와 자동 메모리

Claude Code의 세션은 매번 빈 컨텍스트로 시작하므로, 세션 간 지식 전달을 위해 두 가지 메커니즘을 제공합니다.

| 구분 | CLAUDE.md | 자동 메모리 |
|---|---|---|
| 작성 주체 | 사용자 | Claude |
| 내용 | 지시사항·규칙 | 학습 내용·패턴 |
| 범위 | 프로젝트/사용자/조직 | 저장소 단위(머신 로컬) |
| 로딩 | 매 세션 전체 | 매 세션(첫 200줄 또는 25KB) |

### 7.1 CLAUDE.md 배치 위치

로드 순서는 넓은 범위에서 좁은 범위 순입니다.

| 범위 | 위치 | 용도 |
|---|---|---|
| 관리 정책 | macOS: `/Library/Application Support/ClaudeCode/CLAUDE.md`<br>Linux·WSL: `/etc/claude-code/CLAUDE.md`<br>Windows: `C:\Program Files\ClaudeCode\CLAUDE.md` | 조직 전체 표준·보안 정책 |
| 사용자 | `~/.claude/CLAUDE.md` | 개인 선호(모든 프로젝트) |
| 프로젝트 | `./CLAUDE.md` 또는 `./.claude/CLAUDE.md` | 팀 공유(버전관리 대상) |
| 로컬 | `./CLAUDE.local.md` | 개인 프로젝트 설정(.gitignore 권장) |

작업 디렉터리와 그 상위 디렉터리의 파일은 시작 시 로드되며, 하위 디렉터리의 파일은 해당 디렉터리 파일을 읽을 때 지연 로드됩니다.

### 7.2 작성 원칙

- **분량**: 파일당 200줄 이내를 목표로 합니다. 길어질수록 컨텍스트 소모가 커지고 준수율이 떨어집니다.
- **구체성**: "코드를 잘 포맷하라"보다 "들여쓰기는 2칸을 사용하라"처럼 검증 가능하게 작성합니다.
- **일관성**: 상충하는 규칙이 있으면 Claude가 임의로 선택할 수 있으므로 주기적으로 정리합니다.
- **구조**: 마크다운 헤더와 불릿으로 그룹화합니다.

### 7.3 임포트 문법

```text
See @README for project overview and @package.json for npm commands.

# Additional Instructions
- git workflow @docs/git-instructions.md
```

- 상대·절대 경로 모두 지원하며, 상대 경로는 임포트를 선언한 파일 기준으로 해석됩니다.
- 재귀 임포트는 최대 4단계까지 가능합니다.
- 코드 스팬(백틱)이나 코드 블록 안의 `@경로`는 임포트되지 않습니다.
- 프로젝트 메모리에서 작업 디렉터리 밖을 참조하는 임포트는 최초 1회 승인 대화상자가 표시됩니다.

### 7.4 규칙 파일(.claude/rules/)

```text
your-project/
├── .claude/
│   ├── CLAUDE.md
│   └── rules/
│       ├── code-style.md
│       ├── testing.md
│       └── security.md
```

경로 한정 규칙은 프론트매터로 지정합니다.

```markdown
---
paths:
  - "src/api/**/*.ts"
  - "lib/**/*.ts"
---

# API 개발 규칙
- 모든 API 엔드포인트는 입력 검증을 포함해야 합니다.
- 표준 오류 응답 형식을 사용해야 합니다.
```

`paths`가 없는 규칙은 항상 로드되며, `paths`가 있는 규칙은 해당 패턴의 파일을 읽을 때만 로드됩니다. 사용자 수준 규칙은 `~/.claude/rules/`에 배치합니다.

### 7.5 자동 메모리

Claude가 스스로 기록하는 메모입니다. 유형은 `user`(역할·선호), `feedback`(교정 사항), `project`(진행 중 업무·결정), `reference`(외부 정보 위치)로 구분됩니다.

- 저장 위치: `~/.claude/projects/<project>/memory/`
- 구조: `MEMORY.md`(색인, 매 세션 로드) + 주제별 파일(요청 시 로드)
- 비활성화: `/memory`의 토글, `autoMemoryEnabled: false` 설정, 또는 `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1`
- 저장 위치 변경: `autoMemoryDirectory` 설정(절대 경로 또는 `~/`로 시작)

### 7.6 AGENTS.md 사용 시

Claude Code는 `AGENTS.md`를 직접 읽지 않습니다. 기존 파일을 재사용하려면 다음과 같이 임포트합니다.

```markdown
@AGENTS.md

## Claude Code
`src/billing/` 하위 변경은 플랜 모드를 사용해 주십시오.
```

---

## 8. 스킬(Skills) 만들기

### 8.1 개념

스킬은 반복되는 지시사항·체크리스트·다단계 절차를 `SKILL.md` 파일로 패키징한 것입니다. CLAUDE.md와 달리 **사용될 때만 로드**되므로, 긴 참고 자료도 컨텍스트 비용이 거의 발생하지 않습니다.

> 기존 커스텀 명령어(`.claude/commands/*.md`)는 스킬로 통합되었습니다. 두 방식 모두 `/명령어`로 동작하며 기존 파일은 계속 작동합니다.

### 8.2 저장 위치

| 범위 | 경로 | 적용 대상 |
|---|---|---|
| 엔터프라이즈 | 관리 설정 참조 | 조직 전체 |
| 개인 | `~/.claude/skills/<이름>/SKILL.md` | 모든 개인 프로젝트 |
| 프로젝트 | `.claude/skills/<이름>/SKILL.md` | 해당 프로젝트 |
| 플러그인 | `<plugin>/skills/<이름>/SKILL.md` | 플러그인 활성 범위 |

이름 충돌 시 엔터프라이즈 > 개인 > 프로젝트 순으로 우선합니다. 플러그인 스킬은 `/플러그인명:스킬명` 네임스페이스를 사용하므로 충돌하지 않습니다.

### 8.3 기본 예시

`~/.claude/skills/summarize-changes/SKILL.md`

```markdown
---
description: 미커밋 변경사항을 요약하고 위험 요소를 표시합니다. 사용자가 무엇이 바뀌었는지 묻거나, 커밋 메시지를 요청하거나, diff 리뷰를 요청할 때 사용합니다.
---

## 현재 변경사항

!`git diff HEAD`

## 지시사항

위 변경사항을 2~3개 불릿으로 요약한 뒤, 누락된 오류 처리·하드코딩된 값·갱신이 필요한 테스트 등 위험 요소를 나열해 주십시오.
```

`` !`명령` `` 구문은 **동적 컨텍스트 주입**으로, Claude가 스킬 내용을 읽기 전에 명령을 실행하여 결과를 삽입합니다.

### 8.4 프론트매터 주요 필드

| 필드 | 설명 |
|---|---|
| `name` | 목록에 표시될 이름(개인·프로젝트 스킬에서는 표시용, 명령명은 디렉터리명) |
| `description` | 스킬의 용도와 사용 시점. Claude의 자동 호출 판단 기준(권장) |
| `when_to_use` | 트리거 문구·예시 요청 등 추가 맥락 |
| `argument-hint` | 자동완성 시 인자 힌트(예: `[issue-number]`) |
| `arguments` | 명명된 위치 인자 정의 |
| `disable-model-invocation` | `true` 시 Claude 자동 호출 차단(수동 `/이름` 전용) |
| `user-invocable` | `false` 시 사용자가 직접 호출 불가(Claude 전용 배경지식) |
| `allowed-tools` | 해당 턴 동안 승인 없이 사용 가능한 도구 |
| `disallowed-tools` | 스킬 활성 중 사용 금지 도구 |
| `model` | 스킬 활성 시 사용할 모델 |
| `effort` | 사고 강도(low~max) |
| `context` | `fork` 지정 시 분기된 서브에이전트에서 실행 |
| `agent` | `context: fork`일 때 사용할 서브에이전트 유형 |
| `hooks` | 스킬 호출 시 등록할 훅 |
| `paths` | 자동 활성화를 제한하는 글롭 패턴 |
| `shell` | 인라인 명령 실행 셸(`bash` 기본, `powershell` 지원) |

> claude.ai 업로드·Skills API·`package_skill.py` 경로에서는 `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools` 6개 필드만 허용됩니다. 그 외 필드가 포함되면 업로드가 실패합니다.

### 8.5 라이브 변경 감지

`~/.claude/skills/`, 프로젝트 `.claude/skills/`, `--add-dir` 디렉터리의 `.claude/skills/` 변경은 세션 재시작 없이 반영됩니다. 다만 `SKILL.md` 텍스트에 한정되며, 훅·MCP·에이전트 변경은 `/reload-plugins`가 필요합니다.

---

## 9. 서브에이전트(Subagents)

### 9.1 개념 및 이점

서브에이전트는 격리된 컨텍스트 창에서 특정 작업을 수행하는 전문 보조 에이전트입니다.

- **컨텍스트 보존**: 검색 결과·로그 등 장문 출력을 메인 대화에서 분리합니다.
- **제약 강제**: 도구 제한과 권한 제어를 적용합니다.
- **비용 통제**: Haiku 등 경량 모델로 라우팅합니다.
- **동작 특화**: 목적에 맞는 시스템 프롬프트를 부여합니다.

### 9.2 저장 위치 및 우선순위

| 위치 | 범위 | 우선순위 |
|---|---|---|
| 관리 설정 | 조직 전체 | 1(최상) |
| `--agents` CLI 플래그 | 현재 세션 | 2 |
| `.claude/agents/` | 현재 프로젝트 | 3 |
| `~/.claude/agents/` | 모든 개인 프로젝트 | 4 |
| 플러그인 `agents/` | 플러그인 활성 범위 | 5(최하) |

### 9.3 정의 예시

```markdown
---
name: code-reviewer
description: 코드 품질 리뷰 전문. 코드 작성·수정 후 사용합니다.
tools: Read, Grep, Glob, Bash
model: sonnet
---

당신은 시니어 코드 리뷰어입니다. 발견한 각 이슈에 대하여
1. 문제를 설명하고
2. 현재 코드를 제시하며
3. 개선된 코드를 제안해 주십시오.

가독성, 성능, 모범 사례, 보안을 중심으로 검토하고
critical / warning / suggestion 우선순위로 정리해 주십시오.
```

**필수 필드**: `name`(소문자·하이픈), `description`

**선택 필드**

| 필드 | 용도 |
|---|---|
| `tools` | 허용 도구 목록(미지정 시 전체) |
| `disallowedTools` | 거부 도구 목록 |
| `model` | `sonnet`, `opus`, `haiku`, `inherit` |
| `permissionMode` | `default`, `acceptEdits`, `auto`, `plan`, `bypassPermissions` |
| `maxTurns` | 최대 턴 수 |
| `skills` | 사전 로드할 스킬 |
| `mcpServers` | 사용 가능한 MCP 서버 |
| `hooks` | 검증·로깅용 생명주기 훅 |
| `memory` | 영속 메모리 범위(`user`, `project`, `local`) |
| `isolation` | `worktree` 지정 시 격리된 git 워크트리 사용 |
| `effort` | 사고 강도 |

### 9.4 호출 방법

- **자동 위임**: Claude가 작업 내용과 `description`을 대조하여 자동 위임합니다.
- **@멘션**: `@"code-reviewer (agent)" 인증 변경사항을 봐줘`
- **메인 세션으로 실행**: `claude --agent code-reviewer` 또는 `.claude/settings.json`에 `{"agent": "code-reviewer"}`

### 9.5 내장 서브에이전트

| 이름 | 용도 |
|---|---|
| Explore | 읽기 전용 코드베이스 고속 탐색(기본 Haiku) |
| Plan | 플랜 모드용 리서치 에이전트(읽기 전용) |
| general-purpose | 전체 도구를 사용하는 복합 다단계 작업 |
| statusline-setup 등 | 보조 에이전트 |

특정 에이전트 비활성화는 다음과 같이 설정합니다.

```json
{
  "permissions": {
    "deny": ["Agent(Explore)", "Agent(my-custom-agent)"]
  }
}
```

---

## 10. 훅(Hooks)

### 10.1 개념

훅은 Claude Code의 생명주기 이벤트 시점에 셸 명령·HTTP 호출·MCP 도구·프롬프트를 실행하는 장치입니다. CLAUDE.md가 "권고"라면 훅은 **강제**에 해당하므로, 반드시 수행되어야 하는 검증·포맷팅·차단 로직은 훅으로 구현하시기 바랍니다.

### 10.2 주요 이벤트

| 이벤트 | 시점 |
|---|---|
| `SessionStart` / `SessionEnd` | 세션 시작·종료 |
| `Setup` | CI·스크립트에서의 일회성 준비 |
| `UserPromptSubmit` | 사용자 프롬프트 처리 직전 |
| `UserPromptExpansion` | 슬래시 명령이 프롬프트로 확장될 때 |
| `PreToolUse` | 도구 호출 직전(차단 가능) |
| `PostToolUse` / `PostToolUseFailure` | 도구 호출 성공·실패 후 |
| `PostToolBatch` | 병렬 도구 호출 배치 완료 후 |
| `PermissionRequest` / `PermissionDenied` | 권한 판단 시점 / 자동 모드 거부 시 |
| `SubagentStart` / `SubagentStop` | 서브에이전트 시작·종료 |
| `TaskCreated` / `TaskCompleted` | 작업 생성·완료 |
| `Stop` / `StopFailure` | 응답 종료 / API 오류 종료 |
| `PreCompact` / `PostCompact` | 컨텍스트 압축 전후 |
| `InstructionsLoaded` | CLAUDE.md·규칙 파일 로드 시 |
| `ConfigChange` / `FileChanged` / `CwdChanged` | 설정 변경 / 파일 변경 / 작업 디렉터리 변경 |
| `WorktreeCreate` / `WorktreeRemove` | 워크트리 생성·제거 |
| `Elicitation` / `ElicitationResult` | MCP 서버의 사용자 입력 요청 및 응답 |
| `Notification` / `MessageDisplay` | 알림 발생 / 메시지 표시 |
| `TeammateIdle` | 에이전트 팀 멤버가 유휴 상태로 전환될 때 |

### 10.3 설정 구조

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix",
            "timeout": 600,
            "statusMessage": "린트 검사 중...",
            "async": false
          }
        ]
      }
    ]
  },
  "disableAllHooks": false
}
```

**훅 타입**: `command`(셸), `http`(HTTP 엔드포인트), `mcp_tool`(MCP 도구), `prompt`(경량 모델 판단), `agent`(에이전트 검증)

**종료 코드 규약**

| 코드 | 의미 |
|---|---|
| 0 | 성공. JSON 출력으로 구조화된 제어 가능 |
| 2 | 차단 오류. 해당 동작을 진행하지 않음 |

**경로 플레이스홀더**: `${CLAUDE_PROJECT_DIR}`, `${CLAUDE_PLUGIN_ROOT}`, `${CLAUDE_PLUGIN_DATA}`

### 10.4 훅 배치 위치

| 위치 | 범위 |
|---|---|
| `~/.claude/settings.json` | 모든 프로젝트(로컬 전용) |
| `.claude/settings.json` | 단일 프로젝트(공유 가능) |
| `.claude/settings.local.json` | 단일 프로젝트(로컬 전용) |
| 관리 정책 설정 | 조직 전체(관리자 통제) |
| 플러그인 `hooks/hooks.json` | 플러그인 활성 시 |
| 스킬·서브에이전트 프론트매터 | 해당 실행 중 |

---

## 11. MCP 연동

### 11.1 개념

MCP(Model Context Protocol)는 AI 도구와 외부 시스템을 연결하는 오픈 표준입니다. MCP 서버를 연결하면 Claude Code가 이슈 트래커, 모니터링 대시보드, 데이터베이스, 디자인 도구 등에 직접 접근합니다.

### 11.2 서버 추가

**원격 HTTP 서버**

```bash
claude mcp add --transport http <이름> <URL>
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

**원격 SSE 서버**

```bash
claude mcp add --transport sse <이름> <URL>
claude mcp add --transport sse asana https://mcp.asana.com/sse
```

**로컬 stdio 서버**

```bash
claude mcp add [옵션] <이름> -- <명령> [인자...]
claude mcp add --env AIRTABLE_API_KEY=YOUR_KEY --transport stdio airtable -- npx -y airtable-mcp-server
```

**JSON으로 추가**

```bash
claude mcp add-json example '{"command":"npx","args":["-y","@example/mcp-server"]}'
```

> `.mcp.json`, `~/.claude.json`, `claude mcp add-json`에서 `type` 필드는 `streamable-http`를 `http`의 별칭으로 허용합니다.

### 11.3 설치 범위(Scope)

| 범위 | 플래그 | 적용 대상 |
|---|---|---|
| local | `--scope local` (기본) | 현재 프로젝트, 본인만 |
| project | `--scope project` | `.mcp.json`에 기록, 팀 공유 |
| user | `--scope user` | 모든 프로젝트, 본인만 |

`.mcp.json`의 서버는 팀원이 최초 사용 시 승인 절차를 거칩니다.

### 11.4 관리 명령

```bash
claude mcp list          # 서버 목록 및 연결 상태
claude mcp get <이름>     # 특정 서버 상세 정보
claude mcp login <이름>   # OAuth 인증
```

세션 내에서는 `/mcp`로 연결 상태 확인, 재연결, 활성화/비활성화를 수행합니다.

연결 상태 표기는 `✔ Connected`, `! Needs authentication`, `✘ Failed to connect` 등으로 표시됩니다.

### 11.5 참고사항

- 예약된 서버 이름: `workspace`, `claude-in-chrome`, `computer-use`, `Claude Preview`, `Claude Browser`
- `.mcp.json`에서 환경변수 확장이 지원됩니다.
- MCP 서버는 채널(channel)로 동작하여 외부 이벤트를 세션에 푸시할 수 있습니다.
- MCP 도구가 많아질 경우 도구 검색(tool search) 기능으로 컨텍스트 부담을 줄일 수 있습니다.

---

## 12. 플러그인

### 12.1 개념 및 선택 기준

| 방식 | 스킬 이름 | 적합 용도 |
|---|---|---|
| 표준 구성(`.claude/`) | `/hello` | 개인 워크플로, 프로젝트 전용 설정, 빠른 실험 |
| 플러그인 | `/플러그인명:hello` | 팀 공유, 배포, 버전 관리, 다중 프로젝트 재사용 |

### 12.2 디렉터리 구조

```text
my-plugin/
├── .claude-plugin/
│   └── plugin.json      # 매니페스트 (이 디렉터리에는 이 파일만)
├── skills/              # 스킬 (<이름>/SKILL.md)
├── agents/              # 서브에이전트 정의
├── hooks/hooks.json     # 훅
├── .mcp.json            # MCP 서버 구성
├── .lsp.json            # LSP 서버 구성
├── monitors/            # 백그라운드 모니터
├── bin/                 # PATH에 추가될 실행 파일
└── settings.json        # 기본 설정(agent, subagentStatusLine)
```

> `commands/`, `agents/`, `skills/`, `hooks/`를 `.claude-plugin/` **안에** 두면 인식되지 않습니다. 매니페스트만 그 안에 위치합니다.

### 12.3 매니페스트 예시

```json
{
  "name": "my-first-plugin",
  "description": "학습용 인사 플러그인",
  "version": "1.0.0",
  "author": { "name": "홍길동" }
}
```

### 12.4 개발·테스트·배포

```bash
claude plugin init my-tool                 # ~/.claude/skills/ 아래 플러그인 스캐폴딩
claude --plugin-dir ./my-plugin            # 로컬 디렉터리에서 로드
claude --plugin-dir ./my-plugin.zip        # zip 아카이브 로드
claude --plugin-url https://example.com/plugin.zip
claude plugin validate ./my-plugin         # 배포 전 검증
```

세션 중 변경사항 반영은 `/reload-plugins`로 수행합니다.

### 12.5 마켓플레이스

| 마켓플레이스 | 성격 | 등록 방법 |
|---|---|---|
| `claude-plugins-official` | Anthropic 큐레이션 | 최초 대화형 실행 시 자동 등록 |
| `claude-community` | 커뮤니티 제출·심사 | `/plugin marketplace add anthropics/claude-plugins-community` |

사내 전용 배포가 필요한 경우 비공개 저장소에 마켓플레이스를 호스팅할 수 있습니다.

---

## 13. 권한과 보안

### 13.1 권한 모드

| 모드 | 동작 |
|---|---|
| `default` (Manual) | 각 도구 최초 사용 시 승인 요청 |
| `acceptEdits` | 파일 편집과 `mkdir`·`touch`·`mv`·`cp` 등 일반 파일시스템 명령을 자동 승인 |
| `plan` | 파일 읽기와 읽기 전용 셸 명령만 수행, 소스 편집 금지 |
| `auto` | 백그라운드 안전성 검사를 거쳐 도구 호출 자동 승인 |
| `dontAsk` | 사전 승인된 도구 외 자동 거부 |
| `bypassPermissions` | 권한 프롬프트 생략 |

> `bypassPermissions`는 `.git`, `.claude` 등 보호 경로 쓰기까지 승인 없이 진행하므로, 컨테이너·VM 등 격리 환경에서만 사용하시기 바랍니다. 조직 차원에서 차단하려면 `permissions.disableBypassPermissionsMode` 또는 `permissions.disableAutoMode`를 `"disable"`로 설정합니다.

세션 시작 모드는 `defaultMode` 설정 또는 `--permission-mode` 플래그로 지정합니다.

### 13.2 권한 규칙 문법

기본 형식은 `Tool` 또는 `Tool(specifier)`입니다.

| 규칙 | 효과 |
|---|---|
| `Bash` 또는 `Bash(*)` | 모든 Bash 명령 |
| `Bash(npm run build)` | 정확히 해당 명령만 |
| `Read(./.env)` | 현재 디렉터리의 `.env` 읽기 |
| `WebFetch(domain:example.com)` | 해당 도메인 요청 |
| `Agent(model:opus)` | Opus 모델을 요청하는 에이전트 호출 |
| `Bash(run_in_background:true)` | 백그라운드 실행 Bash 호출 |

파라미터 매칭(`Tool(param:value)`)은 **거부(deny)·질문(ask) 규칙에서만** 사용 가능합니다. 값에는 `*` 와일드카드를 사용할 수 있으며, 지정하지 않으면 정확히 일치해야 합니다.

### 13.3 도구 유형별 승인 정책(Manual 모드 기준)

| 도구 유형 | 예시 | 승인 필요 여부 |
|---|---|---|
| 읽기 전용 | 파일 읽기, Grep | 작업 디렉터리 및 추가 디렉터리 내에서는 불필요 |
| Bash 명령 | 셸 실행 | 내장 읽기 전용 명령 외에는 필요 |
| 파일 수정 | 편집·쓰기 | 필요 |

### 13.4 보안 권고사항

- 조직 표준은 **관리 설정(managed settings)** 으로 배포하여 개별 사용자가 우회할 수 없도록 합니다.
- 기술적 차단은 설정(`permissions.deny`, `sandbox.enabled`)으로, 행동 지침은 관리 CLAUDE.md로 구분하여 관리합니다.
- 신뢰할 수 없는 출처의 플러그인·MCP 서버는 사용하지 않습니다.
- 민감 파일(`.env`, 키 파일 등)은 `permissions.deny`에 명시합니다.
- `/security-review`로 변경 diff의 취약점을 사전 점검합니다.

---

## 14. 설정 파일 체계

### 14.1 우선순위

| 순위 | 계층 | 파일 | 적용 대상 |
|---|---|---|---|
| 1 | 관리 설정 | `managed-settings.json`, MDM, claude.ai 콘솔 | 조직 |
| 2 | 명령행 | `claude --settings` | 해당 세션 |
| 3 | 프로젝트 로컬 | `.claude/settings.local.json` | 본인, 해당 프로젝트 |
| 4 | 프로젝트 공유 | `.claude/settings.json` | 프로젝트 전원 |
| 5 | 사용자 | `~/.claude/settings.json` | 본인, 모든 프로젝트 |

상위 계층에 설정된 키가 하위 계층을 덮어씁니다.

### 14.2 자주 사용하는 설정 키

| 키 | 용도 |
|---|---|
| `permissions.allow` / `ask` / `deny` | 도구 권한 규칙 |
| `permissions.additionalDirectories` | 추가 파일 접근 디렉터리 |
| `permissions.defaultMode` | 기본 권한 모드 |
| `permissions.disableBypassPermissionsMode` | bypass 모드 차단 |
| `hooks` | 훅 정의 |
| `disableAllHooks` | 전체 훅 비활성화 |
| `agent` | 메인 세션으로 사용할 에이전트 |
| `autoMemoryEnabled` | 자동 메모리 활성화 여부 |
| `autoMemoryDirectory` | 자동 메모리 저장 경로 |
| `claudeMd` | 관리 설정에 CLAUDE.md 내용 직접 포함 |
| `claudeMdExcludes` | 특정 CLAUDE.md 로드 제외(글롭) |
| `disableBundledSkills` | 번들 스킬 비활성화 |
| `cleanupPeriodDays` | 세션 기록 보존 기간 |
| `env` | 환경변수 지정 |
| `sandbox.enabled` | 샌드박스 격리 강제 |
| `forceLoginMethod` / `forceLoginOrgUUID` | 로그인 방식·조직 제한 |

> 위 목록은 자주 사용되는 키 위주이며 전체 목록은 아닙니다. 정확한 스키마는 공식 설정 레퍼런스를 확인해 주십시오. 특정 키의 지원 여부가 불확실한 경우 실제 적용 전 검증이 필요합니다. [확인필요]

### 14.3 주요 환경변수

| 변수 | 용도 |
|---|---|
| `ANTHROPIC_API_KEY` | API 키 인증 |
| `CLAUDE_CODE_NEW_INIT=1` | 대화형 `/init` 플로우 활성화 |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY=1` | 자동 메모리 비활성화 |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD=1` | `--add-dir` 디렉터리의 CLAUDE.md 로드 |
| `CLAUDE_CODE_SYNC_SKILLS=1` | claude.ai 계정 스킬 동기화(비대화형 실행 시) |
| `CLAUDE_CONFIG_DIR` | 설정 디렉터리 변경 |
| `CLAUDE_CODE_PROJECT_DIR_NAME` | 프로젝트 디렉터리 이름 지정 |
| `DISABLE_DOCTOR_COMMAND` | `/doctor` 숨김 |

### 14.4 설정 확인 방법

```
/config      # 설정 인터페이스
/context     # 실제 로드된 메모리 파일 확인
/hooks       # 훅 구성 확인
/permissions # 권한 규칙 확인
claude doctor
```

---

## 15. 자동화·스케줄링·CI 활용

### 15.1 스케줄링 수단 비교

| 수단 | 실행 위치 | 특징 |
|---|---|---|
| Routines | 클라우드 | PC가 꺼져 있어도 실행. API 호출·GitHub 이벤트 트리거 지원. 웹·데스크톱 앱 또는 CLI `/schedule`로 생성 |
| 데스크톱 예약 작업 | 사용자 PC | 로컬 파일·도구에 직접 접근 |
| `/loop` | CLI 세션 내 | 프롬프트를 세션 내에서 반복(간단한 폴링) |

### 15.2 CI/CD 연동

- **GitHub Actions**: PR 리뷰, 이슈 트리아지 자동화
- **GitLab CI/CD**: 동일 목적의 파이프라인 구성
- **GitHub Code Review**: 모든 PR에 자동 코드 리뷰 적용
- **인증**: `claude setup-token`으로 장기 OAuth 토큰을 발급하여 CI 환경변수로 주입

### 15.3 헤드리스 자동화 패턴

```bash
# 로그 이상 징후 분석
tail -200 app.log | claude -p "이상 징후가 있으면 알려줘"

# 번역 자동화 후 PR 생성
claude -p "새로 추가된 문자열을 프랑스어로 번역하고 리뷰용 PR을 만들어줘"

# 구조화된 JSON 출력
claude -p --output-format json --json-schema '{"type":"object",...}' "질의"

# 비용·턴 수 제한
claude -p --max-turns 3 --max-budget-usd 5.00 "질의"
```

### 15.4 협업·원격 연동

| 목적 | 수단 |
|---|---|
| 휴대폰·타 기기에서 로컬 세션 이어가기 | Remote Control |
| Telegram·Discord·웹훅 이벤트를 세션에 전달 | Channels |
| 로컬에서 시작해 모바일에서 계속 | `claude --cloud` → 모바일 앱 |
| Slack에서 버그 리포트를 PR로 전환 | Slack 연동(`@Claude` 멘션) |
| 웹 애플리케이션 디버깅 | Chrome 연동 |

---

## 16. 실무 활용 워크플로

### 16.1 프로젝트 온보딩

1. `claude`로 프로젝트 루트에서 세션을 시작합니다.
2. `/init`으로 `CLAUDE.md` 초안을 생성합니다.
3. 생성된 내용에 Claude가 코드로부터 알 수 없는 정보(운영 정책, 이해관계자, 배포 절차)를 보강합니다.
4. 자주 반복하는 절차는 `.claude/skills/`로 분리합니다.
5. 반드시 수행되어야 하는 검증은 훅으로 등록합니다.

### 16.2 대규모 변경

```
/plan  → 계획 수립 및 검토
/batch → 병렬 대규모 변경 실행
/diff  → 변경 내역 확인
/verify 또는 테스트 실행
/code-review → 최종 리뷰
```

### 16.3 컨텍스트 절약 원칙

- 코드베이스 탐색은 Explore 서브에이전트에 위임하여 장문 출력을 격리합니다.
- 긴 참고 자료는 CLAUDE.md 대신 스킬로 분리하여 필요 시에만 로드합니다.
- 파일 유형별 규칙은 `paths` 프론트매터로 조건부 로드합니다.
- 장시간 세션은 `/compact`로 주기적으로 정리합니다.

### 16.4 문서 중심 업무에서의 활용

공공·SI 프로젝트 관리 업무에서는 다음과 같은 활용이 가능합니다.

- 산출물 템플릿을 스킬로 등록하여 문서 형식·문체를 표준화
- 회의록·요구사항 정리 절차를 스킬화하여 일관성 확보
- 조직 표준 문서 규칙(파일명 규칙, 존댓말·보고체 등)을 CLAUDE.md 또는 관리 CLAUDE.md에 명시
- Jira·Confluence·Slack MCP 연동으로 이슈·문서 조회 자동화

---

## 17. 문제 해결(Troubleshooting)

### 17.1 CLAUDE.md 지시가 반영되지 않는 경우

CLAUDE.md는 시스템 프롬프트가 아니라 시스템 프롬프트 뒤의 사용자 메시지로 전달되므로 엄격한 강제력은 없습니다.

점검 순서는 다음과 같습니다.

1. `/context`의 **Memory files** 항목에서 파일이 실제로 로드되었는지 확인합니다.
2. 파일 위치가 로드 대상 경로인지 확인합니다.
3. 지시를 더 구체적으로 수정합니다.
4. 여러 CLAUDE.md 간 상충 규칙이 없는지 확인합니다.
5. 특정 시점에 반드시 실행되어야 하는 사항은 훅으로 전환합니다.

`InstructionsLoaded` 훅을 사용하면 어떤 지시 파일이 언제, 왜 로드되었는지 기록할 수 있습니다.

### 17.2 CLAUDE.md가 너무 큰 경우

- 200줄 초과 시 컨텍스트 소모가 증가하고 준수율이 저하됩니다.
- 4 MiB를 초과하는 파일은 로드되지 않습니다.
- `paths` 기반 규칙으로 분할하거나 `/doctor` 점검의 트림 제안을 활용합니다.
- `@` 임포트는 조직화에는 도움이 되나 컨텍스트 절감 효과는 없습니다.

### 17.3 `/compact` 이후 지시가 사라진 경우

프로젝트 루트 CLAUDE.md는 압축 후 디스크에서 재로드되어 유지됩니다. 하위 디렉터리 CLAUDE.md와 `paths` 규칙은 관련 파일을 읽을 때 재로드됩니다. 대화 중에만 전달한 지시는 유지되지 않으므로, 지속이 필요한 사항은 CLAUDE.md에 기록해 주십시오.

### 17.4 MCP 서버 연결 실패

1. `claude mcp list`로 상태를 확인합니다(`✘ Failed to connect` 등).
2. `/mcp`로 재연결을 시도합니다.
3. 인증이 필요한 서버는 `claude mcp login <이름>`을 실행합니다.
4. 자격 증명 오류 시 실패 상세에 HTTP 상태(예: 401)가 표시됩니다.
5. `claude --debug='mcp'`로 상세 로그를 확인합니다.

### 17.5 플러그인이 동작하지 않는 경우

1. 디렉터리 구조를 확인합니다(`.claude-plugin/` 안에는 `plugin.json`만).
2. 스킬·에이전트·훅을 개별적으로 테스트합니다.
3. `/reload-plugins`를 실행합니다.
4. `/plugin`의 **Errors** 탭을 확인합니다.
5. `claude plugin validate ./플러그인경로`로 검증합니다.

### 17.6 설정이 적용되지 않는 경우

설정 우선순위(관리 > 명령행 > 프로젝트 로컬 > 프로젝트 공유 > 사용자)를 확인하고, `claude --safe-mode`로 커스터마이징을 모두 끈 상태와 비교하여 원인을 좁혀 나갑니다.

---

## 18. 용어 정리 및 참고 링크

### 18.1 용어

| 용어 | 설명 |
|---|---|
| Surface | Claude Code를 사용하는 인터페이스(터미널, IDE, 데스크톱, 웹 등) |
| CLAUDE.md | 세션마다 로드되는 프로젝트·사용자 지시 파일 |
| Auto memory | Claude가 스스로 축적하는 학습 메모 |
| Skill | `SKILL.md`로 패키징된 재사용 가능 절차 |
| Subagent | 격리된 컨텍스트에서 실행되는 전문 보조 에이전트 |
| Hook | 생명주기 이벤트에 연결되는 실행 장치 |
| MCP | 외부 도구·데이터 연결을 위한 오픈 표준 프로토콜 |
| Plugin | 스킬·에이전트·훅·MCP를 묶어 배포하는 확장 패키지 |
| Marketplace | 플러그인 배포 저장소 |
| Permission mode | 도구 실행 승인 방식을 결정하는 모드 |
| Routine | 클라우드에서 실행되는 예약 작업 |

### 18.2 공식 문서 링크

- Claude Code 개요: https://code.claude.com/docs/en/overview
- CLI 레퍼런스: https://code.claude.com/docs/en/cli-reference
- 명령어 레퍼런스: https://code.claude.com/docs/en/commands
- 스킬: https://code.claude.com/docs/en/slash-commands
- 메모리: https://code.claude.com/docs/en/memory
- 서브에이전트: https://code.claude.com/docs/en/sub-agents
- 훅: https://code.claude.com/docs/en/hooks
- MCP: https://code.claude.com/docs/en/mcp
- 플러그인: https://code.claude.com/docs/en/plugins
- 권한: https://code.claude.com/docs/en/permissions
- 설정: https://code.claude.com/docs/en/settings
- 문서 색인(llms.txt): https://code.claude.com/docs/llms.txt

---

## 부록. 빠른 참조 카드

### 가장 자주 쓰는 명령 10선

| 순번 | 명령 | 용도 |
|---|---|---|
| 1 | `claude` | 세션 시작 |
| 2 | `claude -c` | 이전 대화 이어가기 |
| 3 | `/init` | 프로젝트 초기화 |
| 4 | `/plan` | 계획 수립 모드 |
| 5 | `/context` | 컨텍스트 사용량 확인 |
| 6 | `/compact` | 컨텍스트 정리 |
| 7 | `/diff` | 변경 내역 확인 |
| 8 | `/code-review` | 코드 리뷰 |
| 9 | `/memory` | 메모리 파일 관리 |
| 10 | `/doctor` | 설정 점검 |

### 확장 기능 선택 기준

| 상황 | 권장 수단 |
|---|---|
| 매 세션 알아야 할 사실·규칙 | CLAUDE.md |
| 특정 파일 유형에만 적용될 규칙 | `.claude/rules/` + `paths` |
| 반복되는 다단계 절차 | Skill |
| 장문 출력이 발생하는 탐색·조사 | Subagent |
| 반드시 실행되어야 하는 검증·차단 | Hook |
| 외부 시스템 데이터 접근 | MCP |
| 팀·조직 단위 배포 | Plugin |
| 정기 반복 실행 | Routine / 예약 작업 |

---

*본 문서는 2026년 8월 24일 기준 Anthropic 공식 문서를 근거로 작성되었습니다. 제품 업데이트에 따라 명령어와 설정이 변경될 수 있으므로, 중요한 적용 전에는 공식 문서 재확인을 권장드립니다.*
