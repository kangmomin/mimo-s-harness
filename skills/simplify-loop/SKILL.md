---
name: simplify-loop
description: "수정 사항이 없을 때까지 /simplify 반복 실행 (최대 10회)"
user-invocable: true
---

아래 절차를 따라 /simplify 를 반복 실행해:

## 절차

1. `/simplify` 를 실행한다.
2. 리뷰 결과를 확인한다:
   - **코드 수정이 적용된 경우** → iteration 카운트를 1 증가시키고 1번으로 돌아간다.
   - **수정할 사항이 없는 경우** (Applied Changes: 없음, 또는 모든 에이전트가 KEEP 판정) → 루프를 종료한다.
3. **최대 10회** iteration 후에는 수정 사항 유무와 관계없이 종료한다.

## 종료 시 출력

```
Simplify Loop 완료
- 총 iteration: N회
- 총 수정 횟수: M회
```
