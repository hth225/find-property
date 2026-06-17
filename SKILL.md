---
name: find-property
description: Use when a user wants to find and evaluate a South Korean property (전세/월세/매매) and contact a listing agent. Runs an adaptive needs interview, searches Naver 부동산/직방/다방 in the browser, ranks candidates by fit, screens 전세/월세 for 전세 사기 fraud risk, and drafts or sends an inquiry to the agent for a viewing.
---

# find-property

Help a user find the best-fitting South Korean property and arrange a viewing.
Load `reference/korea-real-estate.md` for domain terms, sites, 매매 due-diligence,
and message templates. Load `reference/jeonse-fraud-check.md` before scoring any
전세/월세 listing.

## Phase 1 — INTERVIEW
Ask adaptively, one topic at a time. Branch by 거래유형 (전세/월세/매매) and
건물유형. Separate the answers into:
- **Dealbreakers / hard filters:** budget cap (보증금/월세/매매가), 지역, 최소 방
  수/면적, 입주 가능일, must-have building type.
- **Soft preferences:** 채광/향, 층, 역세권/통근시간, 신축, 주차, 반려동물, etc.
Produce a structured `criteria` object.

## Phase 2 — SEARCH
Search for candidates yourself, following `reference/search-guide.md`: drive the
browser (computer use) across Naver 부동산 / 직방 / 다방 with the `criteria`
facets, normalize the results (incl. 미끼_의심 flags), and pull MOLIT comps where
available. Gather data only here — no ranking yet. If browsing is blocked or
unavailable, ask the user to paste listing URLs and normalize those instead.

## Phase 3 — RANK
1. Drop any candidate violating a hard filter.
2. For each remaining 전세/월세 candidate, run `reference/jeonse-fraud-check.md`
   and attach a badge (🟢/🟡/🔴) with triggered flags.
3. Qualitatively rank survivors, explaining per listing why it fits and where it
   is weak. Present a shortlist. Support iterative feedback ("show me cheaper",
   "closer to the station").

## Phase 4 — CONTACT
For the user's chosen listing(s), draft the Korean inquiry from the template in
`reference/korea-real-estate.md`.
- **Gate 1 (ALL listings):** show the exact message + proposed viewing times and
  send only after the user approves.
- **Gate 2 (🔴 listings):** show the fraud warning with reasons and require
  explicit acknowledgment before contacting.
- Send via computer use through the listing site's contact channel. If computer
  use is unavailable, hand the user the message to send themselves.
- Relay the agent's replies to the user. Never share the user's PII, transfer any
  deposit, or commit to a contract — arrange a viewing only.

## Phase 5 — DUE-DILIGENCE HANDOFF
Before the user signs anything, present the Tier-2 document checklist from
`reference/jeonse-fraud-check.md`. Never declare a listing "safe" from listing
data alone.

## Guardrails
- Use only browser sessions the user authorizes per action; store no credentials.
- If price/전세가율 is uncomputable, say "산정 불가" and treat opacity as risk —
  never fabricate a value or a safe verdict.
