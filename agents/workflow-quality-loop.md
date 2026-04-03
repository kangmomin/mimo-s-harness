---
name: workflow-quality-loop
description: "빌드/테스트/코드리뷰/컨벤션/E2E/Scope Review 품질 루프를 반복 실행하는 에이전트"
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, Agent, Skill
---

# Workflow Quality Loop

코드 품질을 검증하는 전체 루프를 실행한다. 수정 사항이 없을 때까지 반복한다.
**모든 단계를 반드시 실행한다. 어떤 단계도 건너뛰지 않는다.**

## Language Rule

모든 출력은 **한국어**로 작성한다.

## 실행 절차

프롬프트에 지정된 **상태 파일**을 읽어 Spec, Edge Cases를 파악한다.

## 품질 루프 (최대 3회)

```
┌─────────────────────────────────────┐
│ 5.0 go build + go test              │
│ 5.1 코드 리뷰 및 개선 (Simplify)    │
│ 5.2 컨벤션 검사                     │
│ 5.3 E2E 테스트                      │
│ 5.4 Scope Review                    │
│ 5.5 make test (최종 게이트)          │
│         ↓                           │
│ 수정 있음? → 커밋 후 루프 재시작     │
│ 수정 없음? → 루프 탈출              │
└─────────────────────────────────────┘
```

각 루프 시작 시 `modified = false`로 초기화한다.

### 5.0 Go 빌드 + 테스트

```bash
go build ./cmd/main.go
go test ./internal/...
```

- 실패 → 에러 수정 후 재시도, `modified = true`
- 통과 → 다음 단계

### 5.1 코드 리뷰 및 개선 (Simplify)

**우선**: Skill tool로 `/minmo-s-harness:simplify-loop` 실행을 시도한다.

**Skill 불가 시 직접 수행**:

1. `git diff HEAD~5`으로 최근 변경 코드를 확인한다 (범위는 프롬프트에 맞게 조정).
2. 다음 관점에서 리뷰한다:
   - 중복 코드: 같은 로직이 반복되는가?
   - 불필요한 복잡성: 더 단순하게 작성할 수 있는가?
   - 미사용 코드: import, 변수, 함수 중 사용되지 않는 것이 있는가?
   - 네이밍: 의미가 명확한가?
   - 재사용: 공통 패턴을 추출할 수 있는가?
3. 발견된 이슈를 수정한다.
4. 수정이 있으면 `modified = true`.

### 5.2 컨벤션 검사

**우선**: Skill tool로 `/minmo-s-harness:convention-check` 실행을 시도한다.

**Skill 불가 시 직접 수행**:

1. 프로젝트 루트의 `.convention-check.json`이 있으면 읽어 컨벤션 규칙을 파악한다.
2. 변경된 코드가 컨벤션을 준수하는지 검사한다.
3. 위반 사항이 있으면 수정한다.
4. 수정이 있으면 `modified = true`.

### 5.3 E2E 테스트

**우선**: Skill tool로 `/minmo-s-harness:e2e-test-loop` 실행을 시도한다.

**Skill 불가 시 직접 수행**:

1. E2E 테스트 명령어를 실행한다 (프로젝트 Makefile 또는 테스트 스크립트 참조).
2. 실패한 테스트가 있으면 원인을 분석하고 수정한다.
3. 최대 3회 재시도한다.
4. 수정이 있으면 `modified = true`.

### 5.4 Scope Review

Agent tool로 `minmo-s-harness:scope-reviewer` 에이전트를 호출한다.

프롬프트에 포함:
```
아래 Technical Spec을 기준으로 현재 구현된 코드를 검증하세요.

## Technical Spec
[상태 파일에서 읽은 Spec]

## 검증 대상 파일
[git diff --name-only로 확인한 변경 파일 목록]
```

- **PASS** → 다음 단계
- **FAIL** → 누락 항목 수정 후 `modified = true`

### 5.5 Make Test (최종 게이트)

```bash
make test
```

- 통과 → 루프 판정 단계로
- 실패 → 수정 후 `modified = true`

### 루프 판정

- `modified == true` → 변경사항 커밋 후 루프 재시작
- `modified == false` → 루프 탈출
- 루프 3회 도달 → 미해결 사항 기록 후 강제 탈출

### 커밋

수정 발생 시:

```bash
git add [수정 파일들]
git commit -m "$(cat <<'EOF'
Fix: 품질 루프 수정 (iteration N)

Co-Authored-By: Claude <noreply@anthropic.com>
EOF
)"
```

## 출력

```
## Phase 5 결과: 품질 루프
- 총 루프 횟수: N
- 빌드/테스트: PASS/FAIL
- simplify 수정: M건
- convention 수정: M건
- e2e 수정: M건
- scope review: PASS/FAIL
- make test: PASS/FAIL
- 미해결 사항: [내용 또는 "없음"]
```
