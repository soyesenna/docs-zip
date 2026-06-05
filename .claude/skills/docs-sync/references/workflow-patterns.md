# Workflow 스크립트 템플릿

1·3·4단계에서 Workflow 도구에 넘길 스크립트 골격이다. 그대로 복붙하지 말고 대상 제품의 규모(문서 수, 공식 문서 구조)에 맞게 프롬프트와 항목을 조정한다. `args`는 Workflow 호출 시 실제 JSON 값으로 전달한다(문자열로 인코딩하지 않는다).

이 템플릿들은 deep-research 플로우(Scope → Search → Fetch → Verify → Synthesize)를 docs-sync에 맞게 구현한 것이다. 병렬도를 줄이는 방향의 수정(앵글 축소, 검증 표 축소, 항목 묶어서 순차 처리)은 하지 않는다 — ultracode(dynamic workflow)의 병렬 에이전트를 최대한 활용하는 것이 이 스킬의 요구사항이다.

## 사전 리서치 — 빌트인 deep-research 직접 호출

제품 사전 이해가 필요할 때(신규 작성 모드, 모호한 제품명)는 커스텀 스크립트를 짜기 전에 빌트인 named workflow를 그대로 호출한다:

```
Workflow({ name: "deep-research", args: "<제품명> 공식 문서/개발자 문서의 전체 구조, 문서 사이트 URL, 최신 버전, 최근 6개월 주요 변경·신기능·폐기 기능" })
```

커스텀 워크플로 스크립트 안에서 `workflow('deep-research', args)`로 중첩 호출할 수도 있지만, 중첩은 1단계만 허용되므로 메인 루프에서 별도 호출하는 것을 기본으로 한다.

## Workflow 도구 제약사항 (위반 시 스크립트가 죽는다)

- `export const meta = {...}`는 **순수 리터럴**: 변수, 함수 호출, 스프레드, 템플릿 보간 금지
- 스크립트는 **순수 JavaScript**: TS 타입 표기 금지
- `Date.now()`, `new Date()`, `Math.random()` 금지 → 날짜는 메인 루프가 `args.today`로 전달
- `agent()`가 스킵/실패하면 `null` 반환 → 결과는 항상 `.filter(Boolean)` 후 사용
- 워크플로 에이전트가 웹에 접근하려면 프롬프트에 **"ToolSearch로 WebSearch/WebFetch를 로드해서 사용하라"**를 명시해야 한다 (deferred 도구라 로드 없이 호출하면 실패)
- `pipeline()` 안에서 phase를 지정할 때는 전역 `phase()` 호출 대신 `agent()`의 `opts.phase`를 쓴다 (항목별 진행 시점이 달라 race가 생기므로)
- 동시 실행은 자동으로 캡되므로(코어 수 기반) 항목 수는 제한 없이 넣어도 된다

## 공통 스키마

```javascript
const PAGE_SCHEMA = {
  type: 'object',
  required: ['pages'],
  properties: {
    pages: {
      type: 'array',
      items: {
        type: 'object',
        required: ['url', 'title', 'topic', 'importance'],
        properties: {
          url: { type: 'string' },
          title: { type: 'string' },
          topic: { type: 'string', description: '주제 카테고리 (예: installation, cli, config, security)' },
          importance: { type: 'string', enum: ['core', 'secondary', 'minor'] },
          notes: { type: 'string', description: '특이사항 (beta, deprecated, 최근 추가 등)' },
        },
      },
    },
    recentChanges: {
      type: 'array',
      items: { type: 'string' },
      description: '체인지로그/릴리스 노트에서 발견한 최근 변경사항 요약',
    },
  },
}

const AUDIT_VERDICT_SCHEMA = {
  type: 'object',
  required: ['file', 'verdict', 'reasons'],
  properties: {
    file: { type: 'string' },
    verdict: { type: 'string', enum: ['keep', 'update', 'delete'] },
    reasons: {
      type: 'array',
      items: {
        type: 'object',
        required: ['kind', 'detail'],
        properties: {
          kind: { type: 'string', enum: ['outdated', 'wrong', 'missing', 'removed-upstream', 'ok'] },
          detail: { type: 'string', description: '무엇이 어떻게 다른지 구체적으로. update면 이것이 작성 에이전트의 작업 지시서가 된다' },
          sourceUrl: { type: 'string' },
        },
      },
    },
  },
}

const WRITE_RESULT_SCHEMA = {
  type: 'object',
  required: ['file', 'summary', 'sources'],
  properties: {
    file: { type: 'string' },
    summary: { type: 'string', description: 'README 인덱스에 쓸 한 줄 설명 (핵심 키워드 나열형)' },
    sources: { type: 'array', items: { type: 'string' } },
  },
}

const VERIFY_SCHEMA = {
  type: 'object',
  required: ['file', 'passed', 'issues'],
  properties: {
    file: { type: 'string' },
    passed: { type: 'boolean' },
    issues: {
      type: 'array',
      items: {
        type: 'object',
        required: ['severity', 'detail'],
        properties: {
          severity: { type: 'string', enum: ['critical', 'minor'] },
          detail: { type: 'string' },
          sourceUrl: { type: 'string' },
        },
      },
    },
  },
}
```

## 1단계 — Discovery 워크플로

```javascript
export const meta = {
  name: 'docs-discovery',
  description: '제품 공식 문서 전수조사 - 페이지 인벤토리 수집',
  phases: [
    { title: 'Sweep', detail: '멀티앵글 병렬 조사' },
    { title: 'Critique', detail: '누락 영역 검증' },
  ],
}

// args: { product, officialDocsUrl, githubUrl, today }
const PAGE_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }

const COMMON = '먼저 ToolSearch로 WebSearch와 WebFetch를 로드한 뒤 사용하라. '
  + '오늘 날짜: ' + args.today + '. 제품: ' + args.product + '. '
  + '공식 문서: ' + args.officialDocsUrl + '. '
  + '발견한 모든 페이지를 빠짐없이 반환하라. 추측으로 URL을 만들지 말고 실제 확인한 것만.'

const ANGLES = [
  { key: 'sitemap', prompt: '공식 docs 사이트의 sitemap.xml과 robots.txt를 WebFetch로 가져와 문서 페이지 전체 목록을 수집하라. sitemap이 없으면 docs 루트 페이지에서 링크를 재귀적으로 수집하라.' },
  { key: 'nav', prompt: '공식 docs 사이트의 네비게이션(사이드바/목차) 구조를 추출하라. 각 섹션과 하위 페이지를 전부 나열하라. (sitemap에 없는 신규 페이지가 nav에 먼저 뜨는 경우가 있다)' },
  { key: 'changelog', prompt: '체인지로그, 릴리스 노트, 공식 블로그를 찾아 최근 6개월의 변경사항·신기능·폐기 기능을 수집하라. 각 변경이 어느 문서 주제에 영향을 주는지 메모하라.' },
  { key: 'github', prompt: 'GitHub 저장소(' + (args.githubUrl || '웹 검색으로 찾아라') + ')의 README, docs/ 디렉터리, 최근 릴리스를 조사하라. 공식 docs 사이트에 없는 개발자 정보를 수집하라.' },
  { key: 'aux', prompt: '공식 docs 외에 별도 호스팅된 보조 문서(API 레퍼런스 허브, 가이드, 쿡북, 헬프센터)가 있는지 웹 검색으로 찾아 페이지를 수집하라.' },
]

phase('Sweep')
const sweeps = await parallel(ANGLES.map(a => () =>
  agent(COMMON + '\n\n임무: ' + a.prompt, { label: 'sweep:' + a.key, phase: 'Sweep', schema: PAGE_SCHEMA })
))

const valid = sweeps.filter(Boolean)
const byUrl = {}
for (const s of valid) for (const p of s.pages) byUrl[p.url.replace(/\/$/, '')] = p
const pages = Object.values(byUrl)
const recentChanges = valid.flatMap(s => s.recentChanges || [])
log('스윕 완료: 페이지 ' + pages.length + '개, 최근 변경 ' + recentChanges.length + '건')

phase('Critique')
const critique = await agent(
  COMMON + '\n\n다음은 지금까지 수집한 공식 문서 인벤토리다:\n'
  + pages.map(p => '- [' + p.topic + '] ' + p.title + ' — ' + p.url).join('\n')
  + '\n\n이 인벤토리에서 빠진 영역을 찾아라. 공식 문서 사이트를 직접 확인해 인벤토리에 없는 주제/섹션/페이지를 발견하면 반환하라. 없으면 빈 배열을 반환하라.',
  { label: 'completeness-critic', schema: PAGE_SCHEMA }
)
if (critique) for (const p of critique.pages) {
  const k = p.url.replace(/\/$/, '')
  if (!byUrl[k]) { byUrl[k] = p; log('critic이 누락 발견: ' + p.title) }
}

return { pages: Object.values(byUrl), recentChanges }
```

## 3단계 — Audit 워크플로 (업데이트 모드)

메인 루프가 2단계 매핑 결과를 `args.docs`로 전달한다. 문서당 에이전트 하나로 전부 병렬 대조하고, delete 판정은 deep-research Verify 규칙대로 3-vote 적대적 확인을 거친다. pipeline이므로 어떤 문서의 delete 투표가 도는 동안 다른 문서의 audit이 계속 진행된다.

```javascript
export const meta = {
  name: 'docs-audit',
  description: '로컬 문서를 공식 문서와 병렬 대조, delete 판정은 3-vote 적대적 확인',
  phases: [
    { title: 'Audit', detail: '문서당 에이전트 1개 병렬 대조' },
    { title: 'DeleteVote', detail: 'delete 판정당 검증 에이전트 3개 반박 시도' },
  ],
}

// args: { product, dir, today, docs: [{ file, officialUrls: [...], suspectedRemoved: bool }] }
const AUDIT_VERDICT_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }
const REMOVAL_VOTE_SCHEMA = {
  type: 'object',
  required: ['stillExists', 'evidence'],
  properties: {
    stillExists: { type: 'boolean', description: '이 주제가 공식 문서 어딘가에 아직 존재하면 true' },
    evidence: { type: 'string', description: '근거 URL과 한 줄 설명. stillExists가 true면 URL 필수' },
  },
}

const TOOLING = '먼저 ToolSearch로 WebSearch와 WebFetch를 로드하라. 오늘 날짜: ' + args.today + '.\n'

const verdicts = await pipeline(
  args.docs,
  d => agent(
    TOOLING
    + '로컬 문서 ' + args.dir + '/' + d.file + ' 를 Read로 정독하라.\n'
    + '대조 기준 공식 문서: ' + (d.officialUrls.length ? d.officialUrls.join(', ') : '문서 상단의 원문 링크를 따라가라')
    + ' — 전부 WebFetch로 가져와 로컬 문서와 문장 단위로 대조하라.\n'
    + (d.suspectedRemoved
      ? '주의: 이 주제는 최신 문서 인벤토리에서 발견되지 않았다. 정말 공식에서 제거/폐기되었는지 웹 검색으로 직접 확인하라. 단순히 URL이 바뀐 것이면 delete가 아니라 update다.\n'
      : '')
    + '판정 기준: 모든 사실이 최신과 일치하고 핵심 누락 없음 → keep. 틀리거나 구버전이거나 새 내용 누락 → update (reasons에 무엇이 어떻게 다른지 구체적으로 — 이것이 수정 작업 지시서가 된다). 주제 자체가 공식에서 제거됨 → delete (근거 URL 필수).\n'
    + '파일은 수정하지 말라. 판정만 반환하라.',
    { label: 'audit:' + d.file, phase: 'Audit', schema: AUDIT_VERDICT_SCHEMA }
  ),
  // deep-research Verify: "주제가 제거되었다"는 반증 가능한 claim → 3-vote 반박 시도.
  // 증거 우선: 근거 URL 있는 존재 증명이 1표라도 나오면 update로 강등. 3표 전원 제거 확인 시에만 delete 유지.
  (v, d) => {
    if (!v || v.verdict !== 'delete') return v
    return parallel([1, 2, 3].map(n => () =>
      agent(
        TOOLING
        + '주장: "' + args.product + ' 공식 문서에서 [' + d.file.replace('.md', '') + '] 주제가 제거/폐기되었다."\n'
        + '이 주장을 반박하라. 공식 docs 사이트와 웹 검색으로 이 주제를 직접 찾아보라. URL 변경, 섹션 이동, 이름 변경(rebrand)이었을 수 있다. '
        + '확실히 존재하면 stillExists: true와 근거 URL을 반환하라. 정말 제거되었으면 stillExists: false와 그 근거(체인지로그, 폐기 공지 등)를 반환하라.',
        { label: 'delete-vote' + n + ':' + d.file, phase: 'DeleteVote', schema: REMOVAL_VOTE_SCHEMA }
      )
    )).then(votes => {
      const valid = votes.filter(Boolean)
      const refutes = valid.filter(x => x.stillExists)
      if (refutes.length >= 1 || valid.length < 3) {
        // 존재 증명 1표 이상, 또는 표가 모자라면 보수적으로 update 강등
        return Object.assign({}, v, {
          verdict: 'update',
          reasons: v.reasons.concat(refutes.map(r => ({
            kind: 'outdated',
            detail: '삭제 반박됨: 주제가 아직 존재. 새 위치/이름 기준으로 갱신하라',
            sourceUrl: r.evidence,
          }))),
        })
      }
      return v // 3표 전원 제거 확인 → delete 확정
    })
  }
)

const valid = verdicts.filter(Boolean)
log('판정: update ' + valid.filter(v => v.verdict === 'update').length
  + ' / keep ' + valid.filter(v => v.verdict === 'keep').length
  + ' / delete ' + valid.filter(v => v.verdict === 'delete').length)
return valid
```

## 4단계 — Write & Verify 워크플로

수정·신규 작업을 하나의 pipeline으로. 항목별로 작성→3-vote 검증→수정이 독립적으로 흐른다(배리어 없음 — 빠른 항목은 먼저 끝난다). Verify는 deep-research의 3-vote를 렌즈 분산형으로 구현한다: 같은 검증을 3번 반복하면 같은 실수를 3번 놓치므로, 세 에이전트에게 서로 다른 렌즈(accuracy/completeness/sources)를 주고 2/3 다수결로 판정한다.

```javascript
export const meta = {
  name: 'docs-write-verify',
  description: '문서 병렬 작성/수정 후 3-lens 적대적 검증(2/3 다수결), 실패 항목 1회 자동 수정',
  phases: [
    { title: 'Write', detail: '항목당 에이전트 1개 병렬 작성' },
    { title: 'Verify', detail: '문서당 3-lens 검증 패널 (accuracy/completeness/sources)' },
    { title: 'Fix', detail: '검증 실패 항목만 수정' },
  ],
}

// args: { product, dir, today, conventions, tasks: [{ action: 'create'|'update', file, sources: [...], instructions }] }
// conventions: 메인 루프가 references/conventions.md 핵심을 요약한 문자열
// instructions: update면 Audit의 reasons 목록, create면 다룰 주제 범위
const WRITE_RESULT_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }
const VERIFY_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }

const LENSES = [
  { lens: 'accuracy', mission: '틀린 사실, 구버전 정보, 잘못된 파라미터·명령어·엔드포인트·가격·제한값을 찾아 반박하라.' },
  { lens: 'completeness', mission: '출처 공식 문서에 있는 핵심 내용 중 이 문서에 누락된 것을 찾아라.' },
  { lens: 'sources', mission: '출처 공식 문서에 근거가 없는 주장, 문서 상단 원문 링크의 누락·오류를 찾아라.' },
]

const TOOLING = '먼저 ToolSearch로 WebSearch와 WebFetch를 로드하라. 오늘 날짜: ' + args.today + '.\n'

const results = await pipeline(
  args.tasks,
  t => agent(
    TOOLING
    + '출처 공식 문서를 전부 WebFetch로 가져온 뒤 ' + (t.action === 'create' ? '신규 작성' : '수정') + '하라.\n'
    + '대상 파일: ' + args.dir + '/' + t.file + (t.action === 'update' ? ' (Read 후 Edit로 수정. 멀쩡한 부분은 보존)' : ' (Write로 생성)') + '\n'
    + '출처: ' + t.sources.join(', ') + '\n'
    + '작업 내용: ' + t.instructions + '\n\n'
    + '문서 컨벤션:\n' + args.conventions + '\n\n'
    + '중요: 모든 사실은 방금 fetch한 공식 문서에 근거하라. 사전 지식으로 채우지 말라. '
    + '문서 본문을 응답으로 반환하지 말고 파일에 직접 쓰라. 요약과 출처만 반환하라.',
    { label: 'write:' + t.file, phase: 'Write', schema: WRITE_RESULT_SCHEMA }
  ),
  // 3-lens 검증 패널: 문서당 3개 에이전트 병렬, 2/3 다수결 (deep-research Verify)
  (w, t) => w && parallel(LENSES.map(l => () =>
    agent(
      TOOLING
      + args.dir + '/' + t.file + ' 를 Read로 정독하고, 출처 공식 문서(' + t.sources.join(', ') + ')를 WebFetch로 가져와 재대조하라.\n'
      + '[' + l.lens + ' 렌즈] 적대적으로 검증하라: ' + l.mission + '\n'
      + '사소한 표현 차이는 무시하고 실질 문제에 집중하라. 파일은 수정하지 말라.\n'
      + 'critical 이슈가 하나라도 있으면 passed: false.',
      { label: 'verify:' + l.lens + ':' + t.file, phase: 'Verify', schema: VERIFY_SCHEMA }
    )
  )).then(votes => {
    const valid = votes.filter(Boolean)
    const passes = valid.filter(x => x.passed).length
    return {
      file: t.file,
      // 2/3 이상 통과 → 통과. 유효 표가 2개 미만이면 보수적으로 실패 처리해 Fix로 보낸다
      passed: valid.length >= 2 && passes >= 2,
      issues: valid.flatMap(x => x.issues.filter(i => i.severity === 'critical')),
    }
  }),
  (v, t) => {
    if (!v) return null
    if (v.passed) return { file: t.file, ok: true, issues: [] }
    return agent(
      TOOLING
      + args.dir + '/' + t.file + ' 에서 3-lens 검증 패널이 발견한 문제를 수정하라:\n'
      + v.issues.map(i => '- [' + i.severity + '] ' + i.detail + (i.sourceUrl ? ' (근거: ' + i.sourceUrl + ')' : '')).join('\n')
      + '\n각 문제의 근거 URL을 WebFetch로 확인한 뒤 Edit로 수정하라. 문제없는 부분은 건드리지 말라.',
      { label: 'fix:' + t.file, phase: 'Fix', schema: WRITE_RESULT_SCHEMA }
    ).then(r => r ? { file: t.file, ok: true, fixed: v.issues.length, issues: [] } : { file: t.file, ok: false, issues: v.issues })
  }
)

const done = results.filter(Boolean)
log('완료 ' + done.filter(r => r.ok).length + '/' + args.tasks.length
  + (done.some(r => !r.ok) ? ' — 미해결: ' + done.filter(r => !r.ok).map(r => r.file).join(', ') : ''))
return done
```

## 조정 가이드

- **문서가 많은 제품(20개+)**: Write 작업의 `sources`가 페이지 5개를 넘으면 항목을 쪼개라. 에이전트 하나가 너무 많은 페이지를 fetch하면 품질이 떨어진다. 항목을 쪼개는 것은 병렬도를 높이므로 언제나 환영이다.
- **검증 표는 줄이지 않는다**: 3-lens 패널과 delete 3-vote는 deep-research 플로우의 필수 요소다. 사용자가 "빠르게"를 요청해도 유지한다. "더 철저히"를 요청하면 렌즈를 추가한다(예: `reproducibility` 렌즈 — 코드 예제가 실제 공식 예제와 일치하는지).
- **신규 작성 모드**: Audit을 건너뛰므로 tasks는 전부 `action: 'create'`. 목차 설계(2단계)에서 문서별 `sources`와 `instructions`(다룰 주제 범위)를 충실히 채우는 것이 품질을 좌우한다. 제품 이해가 얕으면 먼저 `Workflow({ name: "deep-research" })`를 호출하라.
- **재실행/이어하기**: 워크플로가 중단되면 같은 스크립트로 `resumeFromRunId`를 써서 이어간다. 완료된 agent 호출은 캐시에서 즉시 반환된다.
