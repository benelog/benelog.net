---
allowed-tools: Bash(gh search:*), Bash(gh pr view:*), Bash(gh api:*), Bash(grep:*), Read, Edit
description: content/opensource.md의 오픈소스 기여 표에 최근 PR을 추가하고 통계를 갱신
---

## Context

- GitHub 계정의 최근 PR 목록: !`gh search prs --author benelog --limit 40 --json url,title,createdAt,repository,state --jq '.[] | [.createdAt[0:10], .repository.nameWithOwner, .url, .state, .title] | @tsv'`
- 현재 표에 기록된 PR 링크: !`grep -o 'https://github.com/[^)]*/pull/[0-9]*' content/opensource.md | sort -u`

## Your task

`content/opensource.md`의 기여 표를 최신 상태로 갱신한다.

### 1. 추가할 PR 선별

- 위 Context의 최근 PR 목록과 이미 기록된 링크를 비교해 **표에 없는 PR**을 찾는다.
- 사용자가 URL을 직접 지정했다면 그 PR을 우선 처리한다.
- **외부 오픈소스 프로젝트 기여만** 표에 넣는다. 본인 소유 저장소(`benelog/*`)나 실험용 저장소는 제외한다.
- 사용자가 "최근"이라고만 했다면 최근 몇 건에 그친다. 과거의 누락 항목까지 임의로 소급해 넣지 않는다.

### 2. PR 내용 확인

추측하지 말고 각 PR의 실제 내용을 조회한다.

```
gh pr view <번호> --repo <owner/repo> --json title,state,createdAt,mergedAt,body,files
```

- `files`로 **어디를 고쳤는지** 구분한다: Java 소스 → "Javadoc", `.adoc`/`.md` → "참조 문서", `.xsd` → "XSD", ADR 문서 등. 여러 종류면 "Javadoc과 참조 문서"처럼 함께 적는다.
- `body`와 `title`로 무엇을 고쳤는지 파악한다.

### 3. 행 추가

표 형식: `| 기여일자 | 코드 변경 | 변경 설명 | 기여 성격 | 기여 결과 | 반영 방식 |`

- **기여일자**: PR 생성일(`createdAt`). 표는 날짜 내림차순, 같은 날짜면 생성 시각이 늦은 것이 위.
- **코드 변경**: `[repo#번호](URL)` 형식. `spring-projects/` 소유자 접두사는 생략하고 저장소 이름만 쓴다(다른 소유자와 헷갈릴 때만 `owner/repo#번호`). PR이 아니라 커밋으로 반영된 기여는 `[repo@짧은해시](커밋 URL)` 형식을 쓴다.
- **변경 설명**: 한국어 한 줄. 클래스·프로퍼티 이름 등 식별자는 원문 그대로 둔다.
- **기여 성격**: `버그 수정` / `기능 개선` / `문서 오류` / `문서 개선` 중 하나.
- **기여 결과**: `반영` / `참고` / `기각` / `진행 중`.
- **반영 방식**: `merge` / `메인테이너 커밋` / `리뷰 대기` 등. 릴리스 버전을 알면 `merge (v6.4.0-M2)`처럼 괄호로 덧붙인다. 열린 PR은 `진행 중` + `리뷰 대기`.

같은 성격의 기여는 기존 행과 **표현을 통일**한다. 예를 들어 중복 단어 수정은 모두 `... 중복 단어 표기 수정 ("the the" -> "the")` 형태로 맞춘다. 새 항목에 표현을 다듬었다면 기존 행에도 같은 표현을 소급 적용한다.

### 4. 통계 갱신

본문 상단의 두 줄을 실제 행 수에 맞춰 고친다.

```
총 NN건입니다.
기여 성격별로는 버그 수정 N건, 기능 개선 N건, 문서 오류 N건, 문서 개선 N건입니다.
```

### 5. 검증

수정 후 반드시 실제로 세어 통계와 일치하는지 확인한다.

```bash
grep -c '^| 20' content/opensource.md
for t in "버그 수정" "기능 개선" "문서 오류" "문서 개선"; do
  printf "%s: %s\n" "$t" "$(grep '^| 20' content/opensource.md | grep -c "| $t |")"
done
```

성격별 합계가 총 건수와 맞아야 한다.

### 6. 마무리

문서 수정까지만 하고 멈춘다. 커밋과 푸시는 사용자가 `/push`로 따로 실행한다.
