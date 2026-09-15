# s8-05. 검색 동의어(Search Synonyms) 시스템

**파일 버전:** s8-05
**소속:** 기획1팀
**업데이트:** 2026-09-15
**관련 changelog:** [v2.28](../s2-data/s2-06-changelog.md)

---

## 1. 개요

사용자가 표준 시술명과 다른 표현으로 검색해도 동일한 시술이 노출되도록 하는 동의어(synonym) 매핑 시스템. standard_treatments, encyclopedia 테이블에 search_keywords text[] 컬럼을 추가하고, 검색 함수가 해당 배열까지 매칭하도록 확장.

## 2. 도입 배경

### 문제 사례
- 사용자가 "이마지방제거주사"로 검색 → 결과 없음
- 실제 등록된 표준 시술명: "이마지방**이식**제거주사"
- 부분 문자열 매칭(ILIKE)의 한계: 중간에 "이식" 문자열이 끼어 매칭 실패

### 임상 판단
두 표현 모두 "이마 지방이식 후 과생착된 지방을 제거"하려는 동일 의도이며, 동일 약물로 치료하는 시술. 따라서 별도 항목이 아닌 **동의어**로 처리하는 것이 적절.

## 3. 설계 방식 비교

| 방식 | 장점 | 단점 | 채택 |
|---|---|---|---|
| A. 신규 시술 항목 추가 | 즉시 해결 | 데이터 중복, 의학적 혼란 | ❌ |
| B. 동의어 매핑 (search_keywords) | 원본 무결성, 확장성, 관리 용이 | 초기 구축 필요 | ✅ |
| C. 형태소 기반 토큰화 | 자동 확장 | 오탐 증가, 별도 확장 모듈 필요 | ❌ |

## 4. 구현 내역

### 4.1 스키마 변경

~~~sql
ALTER TABLE standard_treatments ADD COLUMN IF NOT EXISTS search_keywords text[];
ALTER TABLE encyclopedia ADD COLUMN IF NOT EXISTS search_keywords text[];

CREATE INDEX IF NOT EXISTS idx_standard_treatments_search_keywords
  ON standard_treatments USING GIN (search_keywords);
CREATE INDEX IF NOT EXISTS idx_encyclopedia_search_keywords
  ON encyclopedia USING GIN (search_keywords);
~~~

### 4.2 검색 함수 재정의

기존 매칭 로직(공백 제거 후 ILIKE)을 유지하고, search_keywords 배열을 OR 조건으로 추가.

- search_treatments_normalized(q text): treatments LEFT JOIN standard_treatments로 확장. 표준 시술의 키워드까지 매칭.
- search_encyclopedia_normalized(q text): encyclopedia.search_keywords 배열 매칭 추가.

핵심 확장 조건:

~~~sql
OR EXISTS (
  SELECT 1 FROM unnest(COALESCE(search_keywords, ARRAY[]::text[])) kw
  WHERE REPLACE(kw, ' ', '') ILIKE '%' || REPLACE(q, ' ', '') || '%'
)
~~~

### 4.3 최초 등록 동의어

**대상:** 이마지방이식제거주사 (standard_treatments.id = f324c6db-6409-4ec9-aba2-e750ef5a4c21, encyclopedia.id = 160)

**등록 키워드 (8개):**
- 한글 6개: 이마지방제거주사, 이마지방빼는주사, 이마지방녹이는주사, 이마지방분해주사, 이마지방파괴주사, 이마지방융해주사
- 영문 2개: forehead fat removal, forehead fat dissolving injection

## 5. 검증 결과

| 검색어 | 이전 | 이후 |
|---|---|---|
| 이마지방 | ✅ | ✅ |
| 이마 지방 | ✅ | ✅ |
| 이마지방제거주사 | ❌ | ✅ |
| 이마 지방 제거 주사 | ❌ | ✅ |
| 이마지방빼는주사 | ❌ | ✅ |

- SQL 함수 테스트 통과
- 프로덕션 실사이트 확인 완료: https://seoulcp.com/ko/search

## 6. 운영 가이드

### 6.1 신규 동의어 추가

~~~sql
-- 표준 시술에 키워드 추가 (기존 배열에 append)
UPDATE standard_treatments
SET search_keywords = array_cat(
  COALESCE(search_keywords, ARRAY[]::text[]),
  ARRAY['새키워드1', '새키워드2']
)
WHERE id = '<uuid>';

-- 백과사전에도 동일하게 추가
UPDATE encyclopedia
SET search_keywords = array_cat(
  COALESCE(search_keywords, ARRAY[]::text[]),
  ARRAY['새키워드1', '새키워드2']
)
WHERE id = <encyclopedia_id>;
~~~

### 6.2 동의어 등록 원칙

1. **소비자 실제 검색 표현만 등록**: 마케팅 카피나 학술 용어는 지양.
2. **의학적으로 동일한 시술**만 동의어 처리. 유사하지만 다른 시술은 별도 항목.
3. **다국어 지원**: 한글 위주로 등록하되, 영문 표현이 명확한 경우 함께 등록.
4. **띄어쓰기 변형은 등록 불필요**: 검색 함수가 공백을 자동 제거하여 매칭함.

### 6.3 롤백 절차

동의어 개별 제거:

~~~sql
UPDATE standard_treatments
SET search_keywords = array_remove(search_keywords, '제거할키워드')
WHERE id = '<uuid>';
~~~

전체 롤백 (컬럼 삭제, 신중히 사용):

~~~sql
DROP INDEX IF EXISTS idx_standard_treatments_search_keywords;
DROP INDEX IF EXISTS idx_encyclopedia_search_keywords;
ALTER TABLE standard_treatments DROP COLUMN IF EXISTS search_keywords;
ALTER TABLE encyclopedia DROP COLUMN IF EXISTS search_keywords;
-- 이후 검색 함수를 원본 정의로 재생성
~~~

### 6.4 캐시 무효화

검색 페이지는 unstable_cache (60초, tags: ["search"])로 캐싱됨. 동의어 추가 후 즉시 반영이 필요한 경우:
- Vercel Dashboard에서 재배포, 또는
- 60초 경과 대기

## 7. 향후 확장 방침

- 소비자 검색 로그(결과 없음 케이스) 기반으로 신규 동의어 우선순위 결정.
- 시술 카테고리별 일괄 확장은 실제 검색 수요가 확인된 후 진행.
- 관리자 대시보드에 키워드 편집 UI 도입은 백로그 유지.
