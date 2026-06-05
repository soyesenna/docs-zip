# Workflow 스크립트 템플릿

1·3·4단계에서 Workflow 도구에 넘길 스크립트 골격이다. 그대로 복붙하지 말고 대상 제품의 규모(문서 수, 공식 문서 구조)에 맞게 프롬프트와 항목을 조정한다. `args`는 Workflow 호출 시 실제 JSON 값으로 전달한다(문자열로 인코딩하지 않는다).

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

메인 루프가 2단계 매핑 결과를 `args.docs`로 전달한다. 문서당 에이전트 하나, 전부 병렬.

```javascript
export const meta = {
  name: 'docs-audit',
  description: '로컬 문서를 공식 문서와 병렬 대조하여 문서별 판정 산출',
  phases: [{ title: 'Audit', detail: '문서당 에이전트 1개 병렬 대조' }],
}

// args: { product, dir, today, docs: [{ file, officialUrls: [...], suspectedRemoved: bool }] }
const AUDIT_VERDICT_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }

phase('Audit')
const verdicts = await parallel(args.docs.map(d => () =>
  agent(
    '먼저 ToolSearch로 WebSearch와 WebFetch를 로드하라. 오늘 날짜: ' + args.today + '.\n'
    + '로컬 문서 ' + args.dir + '/' + d.file + ' 를 Read로 정독하라.\n'
    + '대조 기준 공식 문서: ' + (d.officialUrls.length ? d.officialUrls.join(', ') : '문서 상단의 원문 링크를 따라가라')
    + ' — 전부 WebFetch로 가져와 로컬 문서와 문장 단위로 대조하라.\n'
    + (d.suspectedRemoved
      ? '주의: 이 주제는 최신 문서 인벤토리에서 발견되지 않았다. 정말 공식에서 제거/폐기되었는지 웹 검색으로 직접 확인하라. 단순히 URL이 바뀐 것이면 delete가 아니라 update다.\n'
      : '')
    + '판정 기준: 모든 사실이 최신과 일치하고 핵심 누락 없음 → keep. 틀리거나 구버전이거나 새 내용 누락 → update (reasons에 무엇이 어떻게 다른지 구체적으로 — 이것이 수정 작업 지시서가 된다). 주제 자체가 공식에서 제거됨 → delete (근거 URL 필수).\n'
    + '파일은 수정하지 말라. 판정만 반환하라.',
    { label: 'audit:' + d.file, schema: AUDIT_VERDICT_SCHEMA }
  )
))
const valid = verdicts.filter(Boolean)
log('판정: update ' + valid.filter(v => v.verdict === 'update').length
  + ' / keep ' + valid.filter(v => v.verdict === 'keep').length
  + ' / delete ' + valid.filter(v => v.verdict === 'delete').length)
return valid
```

## 4단계 — Write & Verify 워크플로

수정·신규 작업을 하나의 pipeline으로. 항목별로 작성→검증→수정이 독립적으로 흐른다(배리어 없음 — 빠른 항목은 먼저 끝난다).

```javascript
export const meta = {
  name: 'docs-write-verify',
  description: '문서 병렬 작성/수정 후 적대적 검증, 실패 항목 1회 자동 수정',
  phases: [
    { title: 'Write', detail: '항목당 에이전트 1개 병렬 작성' },
    { title: 'Verify', detail: '다른 에이전트가 공식 문서와 재대조' },
    { title: 'Fix', detail: '검증 실패 항목만 수정' },
  ],
}

// args: { product, dir, today, conventions, tasks: [{ action: 'create'|'update', file, sources: [...], instructions }] }
// conventions: 메인 루프가 references/conventions.md 핵심을 요약한 문자열
// instructions: update면 Audit의 reasons 목록, create면 다룰 주제 범위
const WRITE_RESULT_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }
const VERIFY_SCHEMA = { /* 위 공통 스키마 붙여넣기 */ }

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
  (w, t) => w && agent(
    TOOLING
    + args.dir + '/' + t.file + ' 를 Read로 정독하고, 출처 공식 문서(' + t.sources.join(', ') + ')를 WebFetch로 가져와 재대조하라.\n'
    + '적대적으로 검증하라: 틀린 사실, 구버전 정보, 출처에 없는 주장, 핵심 누락을 찾아 반박하라. '
    + '사소한 표현 차이는 무시하고 사실 오류에 집중하라. 파일은 수정하지 말라.\n'
    + 'critical 이슈가 하나라도 있으면 passed: false.',
    { label: 'verify:' + t.file, phase: 'Verify', schema: VERIFY_SCHEMA }
  ),
  (v, t) => {
    if (!v) return null
    if (v.passed) return { file: t.file, ok: true, issues: [] }
    return agent(
      TOOLING
      + args.dir + '/' + t.file + ' 에서 검증 에이전트가 발견한 문제를 수정하라:\n'
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

- **문서가 많은 제품(20개+)**: Write 작업의 `sources`가 페이지 5개를 넘으면 항목을 쪼개라. 에이전트 하나가 너무 많은 페이지를 fetch하면 품질이 떨어진다.
- **검증 강화가 필요할 때** (사용자가 "철저히", "꼼꼼히"를 요청): Verify 스테이지를 단일 에이전트 대신 관점 분산 패널(정확성/누락/출처 3렌즈)로 바꾸고 다수결로 판정한다.
- **신규 작성 모드**: Audit을 건너뛰므로 tasks는 전부 `action: 'create'`. 목차 설계(2단계)에서 문서별 `sources`와 `instructions`(다룰 주제 범위)를 충실히 채우는 것이 품질을 좌우한다.
- **재실행/이어하기**: 워크플로가 중단되면 같은 스크립트로 `resumeFromRunId`를 써서 이어간다. 완료된 agent 호출은 캐시에서 즉시 반환된다.
