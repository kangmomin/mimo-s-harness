---
name: db-gen-committed
description: "db-gen으로 migration 파일을 생성하고 자동으로 committed 상태로 변환"
user-invocable: true
---

/db-tools:db-gen 을 실행해서 migration 파일을 생성해.

생성이 완료되면, 생성된 파일의 changeset 라인에서 다음 변환을 자동으로 적용해:
1. `context:draft` → `context:committed` 로 변경
2. `runOnChange:true` 제거
3. comment 에 `[DRAFT]` 접두사가 있으면 제거

즉, 처음부터 committed 상태로 migration 파일을 만들어줘.
