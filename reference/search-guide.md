# Search guide (Phase 2)

How to find candidate listings yourself during the SEARCH phase. There is no
separate search agent — you perform these steps directly using computer use
(browser) and code execution. Gather and normalize data only here; ranking,
fit judgment, and fraud verdicts happen later in Phase 3.

## Input
The `criteria` object built in Phase 1: 거래유형 (전세/월세/매매), 지역, budget
bounds (보증금/월세/매매가), building types, 최소 방 수/면적, and soft-preference
hints.

## What to do
1. Drive the **browser** (computer use) to search Naver 부동산 / 직방 / 다방 with
   the criteria's facets (지역, 거래유형, 가격범위, 면적, 방수, 건물유형). These
   sites are JS-rendered and anti-bot — a plain HTTP fetch will not return live
   listings; use the browser. Requires the user to have Computer use enabled.
2. Where available, pull MOLIT 지역 실거래가 comps for price context (area-level
   only; frequently missing for 빌라/오피스텔 — leave null if so).
3. Flag likely 허위매물/미끼매물: implausibly cheap, stale, duplicated, or
   "이미 계약됨" bait.

## Fallbacks
- If search is blocked / hits a login wall / returns nothing: tell the user
  plainly and ask them to paste listing URLs from the apps, then normalize those.
- If computer use / browser is unavailable: you can only normalize user-pasted
  URLs — say so.

## Output (normalized records — no prose verdicts)
Produce a list of records shaped like:
{ 주소, 거래유형, 보증금, 월세, 매매가, 전용면적, 층/총층, 건물유형, 준공년도,
  관리비, 추정_전세가율(nullable), 시세_comps(nullable), 미끼_의심(bool),
  중개사_연락처, listing_url, 특징[] }

Pass these to Phase 3 (RANK) for filtering, fit ranking, and fraud screening.
