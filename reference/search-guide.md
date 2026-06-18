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
1. Drive the **browser** (computer use) to search **직방 / 다방 / 피터팬의 좋은방
   구하기** with the criteria's facets (지역, 거래유형, 가격범위, 면적, 방수,
   건물유형). These sites are JS-rendered and anti-bot — a plain HTTP fetch will not
   return live listings; use the browser. Requires the user to have Computer use
   enabled.
   - ⛔ **네이버 부동산은 직접 탐색·크롤링하지 않는다.** 네이버파이낸셜 이용약관(제10조)
     과 robots.txt가 자동화 수집을 금지하며, 무단 크롤링은 판례상 법적 리스크가 있다
     (네이버 v. 다윈중개, 2024 네이버 승소). 네이버 매물이 필요하면 사용자가 직접 열람한
     **URL을 붙여넣게 하여** 그것만 정규화한다.
   - ⚠️ **피터팬**은 직거래(중개 없이 임대인 직접 등록) 비중이 높아 공제·검증 장치가
     약하다 → 허위매물 가드레일(아래 3번)을 반드시 적용하고, 직거래 매물은 등기부·
     소유자 본인 확인을 Phase 3에서 강조한다.
2. Where available, pull price-context comps. Cross-check multiple sources since
   any one can be off:
   - **MOLIT 실거래가** (rt.molit.go.kr) — area-level; frequently missing for
     빌라/오피스텔 (leave null if so).
   - **KB시세** — 은행 대출 기준 시세 (아파트·오피스텔에 신뢰도 높음).
   - **안심전세앱** — 빌라·오피스텔 시세 + 임대인 보증사고·체납·악성임대인 이력.
   - 공시지가·과거 경매 낙찰가도 보조 지표로 참고.
   For 빌라/다가구 where 시세 is opaque, the 공인중개사 quote may be 호가, not 시세 —
   treat as advisory. Capture an **경매 낙찰가율** estimate where possible so Phase 3
   can run the 깡통전세 formula (see `jeonse-fraud-check.md`).
3. **허위매물(미끼매물) 가드레일 — 정규화 시 `미끼_의심` 플래그를 적극 부여한다.**
   허위매물은 실제로 없거나 이미 계약된 매물을 미끼로 띄워 연락을 유도하는 수법이다.
   아래 신호 중 하나라도 강하게 걸리면 플래그하고, 둘 이상이면 강한 미끼 의심으로 본다.

   **가격 기반 (핵심):**
   - 주변 시세·동일 단지/동일 면적 comps 대비 **비정상적으로 저렴**(대략 −15~20% 이상
     낮으면 의심). comps는 MOLIT 실거래가 + KB시세로 교차 확인하고, 시세 산정이 불가하면
     "산정 불가 — 검증 필요"로 두고 추측하지 않는다.
   - 보증금만 싸고 관리비가 비정상적으로 높음 → 실질 임대료 은폐(미끼 유형).

   **매물 데이터 기반:**
   - **중복**: 같은 사진/평면도가 다른 중개사·다른 주소로 반복 게시.
   - **stale·재등록**: 오래된 매물이 계속 갱신만 됨, 등록일과 수정일 괴리.
   - **주소·동호수 비공개**하거나 모호, 외부 전경을 가린 사진, 과도하게 보정된/스톡 사진.
   - **확인매물/안심중개사 등 검증 배지 부재** (직거래 매물은 배지 자체가 없을 수 있으니
     배지 유무만으로 단정하지 말고 보조 신호로 사용).

   **연락·행동 기반 (CONTACT 단계에서 관측되면 사후 플래그):**
   - 문의하면 "방금 계약됐다"며 **다른 매물로 유도**(bait-and-switch).
   - 직접 방문/현장 확인을 회피하거나 계약을 재촉.

   **추가 검증 경로 (스스로 조사해 거르기):**
   - 매물 사진을 **역이미지 검색**해 다른 매물/스톡과 중복되는지 확인.
   - **KISO 부동산매물클린관리센터**(네이버·KB 등 연동)에 신고·검증 이력이 있는지,
     사용자에게 신고 메뉴로 검증 요청을 안내.
   - 같은 주소를 여러 플랫폼에서 교차 조회해 가격·정보 불일치를 확인.
   - 로드뷰·지도에서 건물 실재·외관·주변 혐오시설을 대조.

   판정은 advisory다. 미끼 의심은 후보를 즉시 버리기보다 `미끼_의심: true`로 표시해
   Phase 3에서 우선순위를 낮추고 사용자에게 사유를 설명한다.

## Fallbacks
- If search is blocked / hits a login wall / returns nothing: tell the user
  plainly and ask them to paste listing URLs from the apps, then normalize those.
- If computer use / browser is unavailable: you can only normalize user-pasted
  URLs — say so.

## Output (normalized records — no prose verdicts)
Produce a list of records shaped like:
{ 주소, 거래유형, 보증금, 월세, 매매가, 전용면적, 층/총층, 건물유형, 준공년도,
  관리비, 추정_전세가율(nullable), 시세_comps(nullable), 미끼_의심(bool),
  중개사_휴대폰(010, nullable), 중개사_유선(nullable), listing_url, 특징[] }

- 전화번호는 숫자만 남겨 정규화한다(하이픈·공백·`+82` 제거). 매물에 사무실 유선(02·031 등
  지역번호)과 휴대폰(010)이 같이 있으면 **둘을 분리**해 담는다. CONTACT 단계의 sms 딥링크는
  `중개사_휴대폰`(010)만 사용하므로, 010 번호를 놓치지 말고 추출한다(`중개사_유선`은 통화용 보조).

Pass these to Phase 3 (RANK) for filtering, fit ranking, and fraud screening.
