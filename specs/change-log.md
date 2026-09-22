## 2026-09-22 — Done: MVP-T11 + save upsert (usable Confirmed)

- Usable Confirmed 2026-09-22: `37`/AC20, `26`, `27`, `28`, same-`tripId` save upsert, ADR-076 MCP two tools.
- Specs/backlog/plan marked Done; DoD + retrospective gate.

## 2026-09-22 — Change: Save upsert by tripId (`2play-plan-25` AC2)

- Product: 同一行程再点「保存」应覆盖收藏卡，不堆多条历史。
- Spec: Story 25 AC2/AC3、design §2.4.5、ADR-071 D4 — 同 `(userId, tripId)` upsert；无 tripId 仍 create。
- Code: `POST /api/saved` upsert；`@@unique([userId, tripId])` + 迁移折叠重复行。

## 2026-09-22 — MVP-T11 coding: 37 → 26 → 27 → 28 + MCP unregister

- **37 / AC20:** Saved detail isomorphic — constraints panel, panel-head actions (back / export / unsave), PlaceSheet fetches `/api/places/...`.
- **26:** Saved detail readonly chat snapshot from GET `messages` + i18n notes.
- **27:** Replan confirm keeps takeoff/intake, appends divider, re-runs T3 `plan_trip` loop; cancel no-op; does not touch SavedItinerary. Story AC4 patched to `plan_trip` (not Mode H).
- **28:** Client PDF from `ItineraryDto` via `pdf-lib`; Plan + Saved export enabled; missing fields not invented.
- **ADR-076 implemented:** MCP `create-server.ts` registers only `plan_trip` + `fetch_trip_details`; HTTP `/v1` unchanged.

## 2026-09-22 — MVP-T11 docs closeout (no code)

- New [ADR-076](./adr/ADR-076-mcp-public-surface.md): MCP **target** public tools = `plan_trip` + `fetch_trip_details` only; HTTP BFF routes stay; as-built `create-server` still lists legacy tools — unregister is a later story.
- Remaining save work packaged as **MVP-T11**: `37`/24-P1b → `26` → `27` → `28` (next coding = 37).
- Cancelled / Super: `agent-fill-67`, `agent-iconic-69`, `agent-discover-97`, `agent-itinerary-45`/`66`, `2play-plan-41`, 37 whole-pack (AC20 only remains).
- Marked Done in backlog/stories: `38`, `39`, `94a`, `94b` (runtime already shipped).
- **No** application code or test changes in this closeout.

## 2026-09-21 — Done: `2play-plan-25` AC2–3 (Save P2)

- Usable Confirmed 2026-09-21. Save writes `tripId` + Plan assistant-thread messages; GET saved detail prefers `row.tripId` for artifacts.
- Acceptance fix: long-running Next had a Prisma Client from before the column (`Unknown argument tripId` → POST 500). `prisma generate` + restart :3030.
- Saved detail footer: drop sticky `--cloud` bar (`.saved-detail-actions` static + transparent under Travor).
- Next: `2play-plan-37` / 24-P1b, then `26`.

## 2026-09-21 — Implement: `2play-plan-25` AC2–3 (Save P2)

- `SavedItinerary.tripId` + migration; POST `/api/saved` writes tripId (body or session) + Plan assistant-thread messages; new row per save.
- Snapshot stripped to ItineraryDto (no visa/tips). Plan page serializes intake answers + navStatusLines + complete line via `serializePlanSaveMessages`.
- GET `/api/itineraries/[id]` prefers `row.tripId` for artifacts; session dest-match remains fallback (106).
- Status: **Implemented · awaiting usable**. Next after confirm: `37` / `26`.

## 2026-09-21 — Spec lock: `2play-plan-25` AC2–3 (Save P2)

- Rewrite after ADR-071: snapshot = Plan **assistant thread** (not refine chat). Persist optional `SavedItinerary.tripId`.
- `snapshot` stays `ItineraryDto` without visa/tips policy (ADR-046). New row per save; frozen until next explicit save.
- Status: Spec locked · 未编码. UI transcript = 26; isomorphic detail = 37. Next coding: implement 25 AC2–3.

## 2026-09-21 — Drop scenic amenity children from attraction pool

- Extended `ATTRACTION_FRAGMENT_DENY` with nursing room / toilet / locker / clinic templates (CN+EN); `isEligibleAttraction` and `pickNominatedGroundCard` reject cards like `西溪湿地洪园景区母婴室`.
- Satellite dedupe suffixes include the same amenity labels. Destination-agnostic (ADR-042); no Hangzhou catalog; skeleton overlay unchanged.
- Couple-romance Hangzhou case: nominate still lands on parent scenic names, not AMAP amenity children.

## 2026-09-21 — Return hydrate artifacts (`2play-plan-106` Done)

- `refreshItineraryFromTripLedger` fetches `artifacts` with skeleton/filled; maps via `travelTipsPayloadFromSlice`.
- `GET /api/plan/current` returns sibling `travelTips` (never written into `itineraryJson`).
- Plan page hydrate calls `setTravelTips`. Saved detail hydrates tips when session trip destination matches.
- Usable Confirmed 2026-09-21. Knowledge: `plan-return-hydrate-artifacts.md`. No new ADR (ADR-046/073/075).
- Next: Save P2 (`25` / `37` / `26` …).

## 2026-09-21 — Visa batch accepted: 94c + 107 + card 01 title

- **`2play-plan-94c`:** Done usable Confirmed — quota / missing nationality / unknown dest degrade honestly.
- **`2play-plan-107`:** Done usable Confirmed — same-ISO hide; show visa_free; Greater China pairs separate; no invented arrival card.
- Tips card 01 title: `play.plan.travel_tips_visa` →「目的地」/ Destination (mock + locales).
- Next coding story: `2play-plan-106` artifacts hydrate on return.

## 2026-09-21 — Same-ISO visa hide + show visa-free (`2play-plan-107` Done)

- Skip `visa_requirement` when passport alpha-3 equals destination. Slice and panel hide same-ISO / `not_applicable`.
- `visa_free` (CHN→SGP) still shows; link text includes requirement (免签 N 天). CHN→HKG `special` still shows.
- No SG Arrival Card copy. Usable confirmed 2026-09-21 with 94c.

## 2026-09-21 — Visa honest degrade (`2play-plan-94c` Done)

- Missing destination country: no `visa_requirement` call; visa link and notice stay hidden.
- Missing passport nationality with a known destination country: no Orizn call; card 01 shows an i18n notice and a link to `/profile`.
- Quota / unconfigured / provider failure: tips carry `play.plan.travel_tips_visa_unavailable` only. Requirement text and visa-free days are not shown.
- Usable confirmed 2026-09-21 with 107.

## 2026-09-21 — Tips panel during fill (`2play-plan-90e` Done)

- Plan page mounts the tips panel (loading) when the T3 skeleton is on screen, before the fill hold.
- Fill does not await `visa_requirement` before the first artifacts read. After each filled stop, BFF polls `artifacts` and yields NDJSON `tips` when the payload changes. Late fetch remains a safety net. [ADR-075](./adr/ADR-075-tips-ndjson-via-fill-poll.md).
- Usable confirmed 2026-09-21. `2play-plan-106` (hydrate artifacts on return) is still not started.

## 2026-09-21 — Visa popover opens downward and stays in the viewport

- Default placement is below the visa link. Height is clamped to remaining viewport space; flip above only when below is under ~12rem and above is larger.
- Requirement title stays in a sticky head while facts and lists scroll. Extension copy is not repeated when details already include the phrase.

## 2026-09-21 — Visa popover canonical applied

- `06-plan.html` and Plan card 01 use `.travel-tips-popover--roomy` (wider, taller, scrollable). Tips grid/panel overflow visible so the layer is not clipped.
- 90e / 106 still not implemented.

## 2026-09-21 — Visa 详情 canonical = 加大悬浮层

- 用户选定 [`06-plan-visa-popover.html`](./2play-specs/ui-mockup/06-plan-visa-popover.html)。94b / §3.5.6 真源为 hover/focus 可滚动 popover（`--roomy`，卡面不裁切）。
- 并入 `06-plan.html` 与 where2play CSS/DOM 待 Agent 模式执行。90e / 106 仍未编码。

## 2026-09-21 — Visa mock 四案；tips 同步推送与回访 hydrate（spec only）

- **Visa UI：** 对比 mock [`06-plan-visa-popover.html`](./2play-specs/ui-mockup/06-plan-visa-popover.html) / [`expand`](./2play-specs/ui-mockup/06-plan-visa-expand.html) / [`drawer`](./2play-specs/ui-mockup/06-plan-visa-drawer.html) / [`dialog`](./2play-specs/ui-mockup/06-plan-visa-dialog.html)。[`06-plan.html`](./2play-specs/ui-mockup/06-plan.html) 真源未改；点名后再改 94b 运行时。
- **Tips 同步（方案 B）：** 骨架上主区即 mount 贴士区；visa 不挡 tips 首包；agent dualWrite `artifacts.tips`/`visa` 后推 NDJSON `tips`。90d late-fetch-only 不再是唯一路径。运行时未改。
- **回访丢失：** 根因是 `GET /api/plan/current` 只拉 skeleton/filled + React `travelTips` 未水合。锁定：hydrate 必须 fetch `artifacts`（ADR-046），不把政策写入 `itineraryJson`。运行时未改。

## 2026-09-21 — Visa popover shows all honest Orizn fields (`2play-plan-94b`)

- Plan card 01 popover maps documents, process, processing time, cost, validity, max stay, extension, embassy/transit when present, plus source URL and verified date.
- `{upgrade:…}` / empty fields are omitted; no invented policy (ADR-046).
- Agent mapper persists `cost` (and embassy/transit strings) when they are real text.

## 2026-09-21 — Implement `2play-profile-38` (nationality required) and `2play-plan-94a` (visa write)

- **38:** Register and profile nationality is required (`play.errors.nationality_required`). Invalid alpha-3 still rejected. Legacy null rows are not wiped until the user saves a code.
- **94a:** Geocode hits include `country_code` (Google `short_name`; AMAP 中国 → `CN`). Fill writes `artifacts.visa` via `visa_requirement` when session nationality and destination alpha-3 are both known; otherwise the call is skipped (94c). UI popover is not in this batch (94b).
- **Tests:** register/profile vitest; geocode-hit; plan-skeleton-fill 94a write/skip. Fixture path only.

## 2026-09-21 — Visa 94a design locked; nationality required

- **Design §3.5.6:** write path = BFF `visa_requirement` + `trip_id` + fetch `artifacts.visa`. Dest = geocode `country_code` alpha-2 → alpha-3 via `PASSPORT_COUNTRIES` (not a city table). Display = `06-plan.html` card 01 (94b later).
- **38:** register/profile nationality **required** (`play.errors.nationality_required`).
- **Order:** 38 then 94a. 94b/94c not in this batch.

## 2026-09-21 — Visa stories: nationality required; 94a/b/c in plain language

- **38:** 注册/资料国籍 **必填**（原选填作废）；旧账号空国籍由 94c 提示补填。
- **39 / 94b:** 视觉真源锁定 [`06-plan.html`](./2play-specs/ui-mockup/06-plan.html) 贴士卡 01（链接 + popover + 数据来源）；`10-travel-advice.html` 不阻塞 MVP。
- **94 split:** 94a = 后台写入；94b = 按 mock 画出；94c = 失败诚实。无运行时代码。


- **Priority:** Visa first (user confirmed); save 25/26 deferred to **P2**.
- **Backlog/plan:** §0 / §8 当前下一步 = `2play-plan-39` → `2play-profile-38` → `2play-plan-94a` → `94b` → `94c`.
- **Stories:** `2play-plan-94` epic 拆 US94a (BFF write) / US94b (Plan UI) / US94c (honest degrade); Plan 贴士区为主战场.
- **Code:** none this change (specs only). Agent F48/F76 remain Done(producer).
- **Shipped earlier same day:** tips/geocode commits `0e1a112` (specs) · `c2ebf05` (agent) · `6688414` (where2play).

## 2026-09-20 — DoD: tips 90d/93d + dest geocode-114 Done

- **Usable:** 用户确认可用（杭州 tips 留存与 locale；鼓浪屿→厦门市）。
- **Retrospective:** knowledge 已有 `tips-prose-locale-weather-drivers.md`、`dest-geocode-city-or-scenic.md`；**无新 ADR**。
- **Next (updated):** Visa track P0；保存闭环 P2。

## 2026-09-20 — Dest geocode: city or scenic only (`agent-geocode-114`)

- **Bug:** 鼓浪屿 → AMAP geo[0] 西宁住宅区 → 中国/西宁市；discover 锚错城。
- **Fix:** AMAP geocode skip residential/road levels; place/text scenic fallback → parent city; empty `city: []` handled.
- **Tests:** amap direct + geocode-hit; knowledge `dest-geocode-city-or-scenic.md`. No city encyclopedia.

## 2026-09-20 — Fix tips-prose CN mix (`drizzle` in clothing)

- **Cause:** `weatherContext` injected English `WeatherDriver` enums (`drizzle`) into tips-prose LLM prompt; model echoed them in `clothing`.
- **Fix (agent):** localized `weatherContextForTips` + `itinerary.weather.driver_*` catalogs; CN/HK/TW validation retry; overlay forbid-list.
- **Tests:** `travel-tips.test.ts` drizzle-context + retry; stories 93d US6 / 90d US4; knowledge note.

## 2026-09-20 — Fix Hangzhou fill: tips vanish after done

- **Cause:** fillOnly fetched `artifacts.tips` only at fill start (120ms retry); Hangzhou tips LLM finished during fill; `finally` cleared loading with `travelTips` still null and unmounted the panel.
- **Fix:** refetch artifacts after last `day_done` and yield `tips` before `done`; keep panel with `{}` if still empty.
- **Test:** `should_yield_tips_after_fill_when_artifacts_arrive_late`.

## 2026-09-20 — Implement `2play-plan-90d` (four-card tips UI)

- **Agent:** `skeleton_only` fire-and-forget `artifacts.tips` (no visa).
- **BFF:** fillOnly `fetch` artifacts → NDJSON `tips` (no `travelTips` write on fill).
- **UI:** destination card title i18n; empty states; visa hidden unless label; panel on planning/done.
- **Tests:** plan-travel-tips / plan-skeleton-fill / plan-fetch-trip / agent skeleton_only tips.

## 2026-09-20 — Docs: tips UI vs visa backlog visibility

- **Next:** `2play-plan-90d`（AC Ready）四卡 UI；**visa 另条** `2play-plan-94`（不在 90d）。
- **Index:** §0.1 新增 24-P2d；§1 / stories 对齐 90d GWT + 94 stub；`agent-tips-70` 并入 90d。

## 2026-09-20 — Implement `agent-tips-93d` (tips-only artifacts)

- **Code:** `plan_trip` starts `travelTips` after skeleton (parallel with fill); dualWrite `artifacts.tips` only; strip visa from agent + legacy full loops.
- **Tests:** `plan-trip.test.ts` describe `agent-tips-93d` (TC-T10-93d-01..05) green.
- **Specs:** stories/test-plan/backlog/plan → Implemented（usable 待确认）.

## 2026-09-20 — Specs: lock `agent-tips-93d` (tips-only, AC Ready)

- **Contract:** `plan_trip` after `skeleton_ready` dualWrites `artifacts.tips` only; grounded iconic from skeleton; parallel with fill; no visa; no 2play UI.
- **Docs:** agent-stories GWT · agent-design §5 · TC-T10-93d-01..05 Planned · backlog/plan strip visa from 93d.
- **No production code.** Visa → `2play-plan-94`. UI → `2play-plan-90d`.

## 2026-09-20 — Docs: explode MVP-24 stories into §0.1 / plan §9

- **Backlog:** [`product-backlog.md`](./product-backlog.md) §0.1 lists every 24-P0…P5 story with Super / T10 / Paused / Cancelled.
- **Plan:** [`plan.md`](./plan.md) §8 T10 ordered list; §9 remaining Paused only.
- **No code.** No new ADR.

## 2026-09-20 — Usable Confirmed: ADR-071, agent-meal-117, agent-meal-118

- **Confirm:** product usable 2026-09-20. Stories **Done**. Next: MVP-T10.
- **Retrospective:** no new ADR. Knowledge: [`meal-118-type-bayes.md`](./knowledge/agent/meal-118-type-bayes.md) HK 7 Paintings is restaurant/dinner-show (type gate correctly allows).
- **Git:** 117/118/071 code already on origin; this is specs status only.

## 2026-09-20 — City-state geocode: HK/MO takeoff verify

- **Bug:** `香港` live geocode 有 `country`、无 `city` → 2play `destVerified` 失败 →「无法核实该目的地」。
- **Fix:** `parseGoogleAddressComponents` / AMAP admin：无 locality 时 `city = country`；`formatDestVerifiedLabel` 城邦折叠为单名（可带 `city_en`）。
- **Tests:** geocode-hit + plan-dest-label. Orthogonal to ADR-052 two-region routing.
- **Docs:** `agent-geocode-100` AC2.

## 2026-09-20 — ADR-052 two-region: mainland AMAP / elsewhere Google-only

- **Code:** `DestinationRegion` = `mainland | other`; HK/MO detect before CN bbox; strip 香港/澳门 from `china-cities`.
- **Behavior:** omit `providers[]` → 大陆 AMAP-only；港/澳/台/海外 Google-only + Tripadvisor enrich. Explicit `providers[]` still overrides. Takeoff HK city-missing unchanged (orthogonal).
- **Tests:** provider-resolver + plan-trip HK/MO Google-only expectations.
- **Docs:** ADR-052 D2/D3；ADR-058 note；vendor-adapters；places-agent-loop；agent-design 策略表；agent-stories US5/US5b；TC-T1-routing.
- **No new ADR.** ADR-058 remains for explicit dual-source / legacy pool rows.

## 2026-09-20 — Implemented: `agent-meal-118` type gate + Bayesian rank

- **Code:** `isGoogleMealTypeAllowedForQuery` + `mealRankScore` (m=50, C=4.0); Nearby no longer overwrites `category` with includedTypes.
- **Tests:** TC-M118-01..06 vitest. No new ADR. No hours gate.
- **Docs:** stories/test-plan/plan/backlog; status Implemented pending usable confirm.

## 2026-09-20 — Specs: `agent-meal-118` type gate + Bayesian rank (no code)

- **Locked:** Google meal type = `primaryType` else `types[0]`; restaurant query allows `restaurant` / `*_restaurant` except breakfast/cafe/bar/bakery; rank `score` m=50 C=4.0; own story after 117 usable.
- **Docs:** agent-design §4.2 · stories/test-plan TC-M118 Planned · plan 临时 7 · [`meal-118-type-bayes.md`](./knowledge/agent/meal-118-type-bayes.md).
- **No production code.** No new ADR.

## 2026-09-20 — Specs: `agent-meal-117` plan confirmed; DoD pending usable

- **Contract:** [`agent-design.md`](./agent-specs/agent-design.md) §4.1 confirmed (A+C+B, no new ADR).
- **Verify:** places-agent vitest `plan-next-stop` / `live` / `direct` — 84 passed (TC-M117).
- **Plan:** [`meal-117-dev-plan.md`](./knowledge/agent/meal-117-dev-plan.md) steps 1–5 Done; step 6 usable confirm pending. No commit until usable yes.

## 2026-09-20 — Specs: lock `agent-meal-117` A+C+B design (no code)

- **Contract:** [`agent-design.md`](./agent-specs/agent-design.md) §4.1. Also fill-resolve-meals, latency note (historical vs target), vendor-adapters dining row.
- **Plan (draft):** [`meal-117-dev-plan.md`](./knowledge/agent/meal-117-dev-plan.md). No new ADR. DoD usable still pending.

## 2026-09-20 — Implemented: Google fill 搜餐墙钟（`agent-meal-117`）

- **A** stop after first gated centroid; **C** `searchRestaurants` timeout skips Worker MCP; **B** generic dining uses `searchNearby`.
- **Tests:** TC-M117-01..07 vitest. Probe: Taipei fill_s 102→66s; Lisbon 118→78s (retry after skeleton LLM fail).
- **Knowledge:** [`meal-117-lisbon-taipei-probe.md`](./knowledge/agent/meal-117-lisbon-taipei-probe.md).

## 2026-09-20 — Implemented: fill 时无LLM按规则排餐（`agent-meal-116`）

- **Code:** `pickMealVenue` / quality gate in `meal-corridor.ts`; `resolveMealVenue` returns `{ card, lowSignal }`; fill note `meal_low_signal`; Google `userRatingCount` → `user_ratings_total` + `types[]` fieldMask.
- **Tests:** TC-M116-01..08 + fill note + mapper; `no-city-hardcode` green. No new ADR.
- **Docs:** stories/test-plan/backlog/plan; probe note [`meal-116-hz-taipei-probe.md`](./knowledge/agent/meal-116-hz-taipei-probe.md).
- **DoD:** usable Confirmed 2026-09-20.

## 2026-09-20 — Map quota restored (AMAP + Google)

- **Operator:** AMAP + Google Maps quota **restored** (2026-09-20). P0 restore-quota ToDo **Done**.
- **ADR-071:** live browser verify **unblocked**, not yet run; DoD usable confirm still pending.
- **Specs:** [`plan.md`](./plan.md) · [`product-backlog.md`](./product-backlog.md) §0 · [ADR-071](./adr/ADR-071-descope-in-page-refine-replan-only.md) § Verification.

## 2026-09-20 — Specs: fill 时无LLM按规则排餐（`agent-meal-116`）

- **Chosen** over 骨架LLM搜餐. ADR-049 D3 unchanged; ADR-074 stays Proposed.
- **Rules:** `MEAL_RATING_MIN=3.5`; Google `user_ratings_total>=20` when present; drop `cafeteria`/`food_court`; AMAP rating-only; unrated never champion if a gated card exists; 5km all-low still pick best + `meal_low_signal`. No name/city deny lists.
- **Docs:** [`research_fill_rule_meals.md`](./knowledge/agent/research_fill_rule_meals.md) · stories/design/test-plan TC-M116. **No production code this pass.**

## 2026-09-20 — Chosen: fill 时无LLM按规则排餐

- Named + selected over 骨架LLM搜餐. Notes: [`research_fill_rule_meals.md`](./knowledge/agent/research_fill_rule_meals.md). Design in discussion; no code.

## 2026-09-20 — Named research: 骨架LLM搜餐

- **Knowledge:** [`knowledge/agent/research_restaurant_pre_search.md`](./knowledge/agent/research_restaurant_pre_search.md) — 持环模型搜餐 grounding，骨架只抄 id；否决「只改提示词」与代码簇搜。未实施。

## 2026-09-20 — Skeleton meals vs fill eval (ADR-074 Proposed)

- **Decision lock:** If ADR-049 D3 is revised, **only variant A** (grounded meal ids at skeleton; fill still Directions). **Reject B** (model/heuristic transit). **C** equals fill-before-paint and requires an explicit ADR-063 supersede.
- **Knowledge:** [`knowledge/agent/skeleton-meals-vs-fill-eval.md`](./knowledge/agent/skeleton-meals-vs-fill-eval.md).
- **ADR:** [ADR-074](./adr/ADR-074-skeleton-meals-variant-a-only.md) Proposed — not implemented; meal slots stay fill-search until a follow-up story.
- **Live baseline (2026-09-20):** Hangzhou ready 15/15, skeleton **13.7s**, fill **15.4s**, total **55s**. Taipei ready 18/18, skeleton **22.5s**, fill **67.8s**, total **131s**. Eval note records trip_ids.

## 2026-09-20 — Temp 5: list thumbs from fill `photos[0]` (ADR-051 D6)

- **Root cause (class):** `slimCandidatesForStore` kept only raw `https://` strings, dropping AMAP `http://*.autonavi.com` before fill; timeline read empty `photos[]` while place-sheet used live Details.
- **Agent:** slim uses `pickDisplayablePhotoUrl` (AMAP upgrade + placeholder reject); fill keeps same-provider `resolveDisplayPhoto` / Details when pool has no displayable photo.
- **Tests:** `trip-store.test.ts` AMAP http + example.com; `plan-next-stop-stay-photo.test.ts` Temp 5 pool + Details + tip-id name-search photo. Results: [`knowledge/agent/temp5-list-thumbs-test-results.md`](./knowledge/agent/temp5-list-thumbs-test-results.md).
- **Follow-up (平湖秋月 live):** AMAP inputtips id `B023B024F8` 详情 404；fill 在 Details 空时按站名搜索只抄 `photos[0]`，不换 pointer。
- **2play:** no list fetch (ADR-051 D2); existing `firstHttpPhoto` display fallback unchanged.

## 2026-09-19 — Resolvable native_id + displayable photo gate (verify_* / example.com)

- **Root cause (Lisbon live):** Geo-merged `AttractionPoi` kept POC `verify_*` cards with `cdn.example.com` photos; skeleton exact-name attach bound `Torre de Belém` / `Mosteiro dos Jerónimos` to harness ids — list thumbs 404 while sheet could still load via details.
- **Agent:** `registrableNative` / `listPoisForDestination` skip unverifiable ids; `poolNativeIdIndex` / `pointerFromCard` / `dropUnknown` ids; `isDisplayablePhotoUrl` rejects RFC 2606 placeholder hosts; script `scripts/purge-unverifiable-pois.ts`.
- **2play:** `itinerary-skeleton-map` `firstHttpPhoto` skips placeholder hosts (list `photoUrl`).
- **Tests:** TC-F115-01, registry skip verify, `resolve-display-photo.test.ts`, where2play skeleton-map placeholder test.

## 2026-09-19 — Temp 2: skeleton attraction pool pointer gate (ADR-072 D2)

- **Agent:** `validateSkeleton` rejects attractions without pool `(provider, native_id)`; `dropAttractionsWithoutPoolPointer` after attach in `make_itinerary`; attach keeps only pool-matched ids.
- **Tests:** TC-F114-01..02 (`make-itinerary.test.ts`).
- **Queue:** 临时 5 杭州 list 缩略图写入 `plan.md` / `product-backlog.md` §0（本 PR 不实现 fill photo / e2e）。

## 2026-09-19 — Plan session draft persist (ADR-073 / 2play-plan-105)

- **Fix:** `GET /api/plan/current` keeps filled `PlanSessionCache.itineraryJson` on refresh; skeleton fetch no longer wipes board after 我的行程 → 行程规划.
- **ADR:** [ADR-073](./adr/ADR-073-plan-session-draft-itinerary.md). Save still `SavedItinerary`; next plan/Replan overwrites draft.
- **E2E:** `where2play/e2e/e2e_draft_persist_hz.py` · `make test-e2e-draft-persist` (杭州 / AMAP).

## 2026-09-19 — Google meal latency knowledge + temp plan #3

- **Knowledge:** [`google-restaurant-search-latency.md`](./knowledge/maps/google-restaurant-search-latency.md) — AMAP around vs Google searchText + corridor × points + MCP retry.
- **Plan:** 临时第 3 项搜餐超时引用该备忘；第 1 项助手线程仍待方案确认。

## 2026-09-19 — Clarify T3 SoT + track Google meal latency

- **核实：** T3 行程真源仍是 Trip Store + `fetch_trip_details`（ADR-046）。BFF `POST /api/plan/trip` 在 skeleton_only ready 后读库切片；不是用 `plan_trip` 写信封当账本。浏览器一次 JSON 等待是进度 UX，不是架构回退到整包 JSON。
- **ToDo：** Google 排餐墙钟列为临时队列下一项（`plan.md` / `product-backlog.md` §0）。

## 2026-09-19 — Soft two-step assistant thread (skeleton + fill_begin + fill)

- **UX:** After T3 framework, keep skeleton spine; show `play.plan.assistant_fill_begin`; wait ~2s; stream fill. Fill spine **appends** — does not replace the skeleton message.
- **Code:** `where2play` `SKELETON_HOLD_BEFORE_FILL_MS` · `fill_begin` thread kind.
- **Specs:** `2play-design.md` §4.11 / T3 B3 i18n table.

## 2026-09-19 — ADR-072 Accepted: fill by `(provider, native_id)`

- **决定：** 景点身份 = `(provider, native_id)`；骨架复制指针；fill 抄池卡；无指针时 search 与池 id 求交（非 `searched[0]`）；Google 显示名 fill 时 UI `languageCode` Details 写一次；AMAP 不译；性能非 AC。
- **ADR：** [ADR-072](./adr/ADR-072-stop-identity-provider-native-id.md) **Accepted**；[ADR-052](./adr/ADR-052-map-provider-routing.md) D9 补充；[ADR-042](./adr/ADR-042-no-city-encyclopedia-in-source.md) 缩略图 alias 条款。
- **故事/测试：** `agent-fill-113` · TC-F113-01..05；`agent-quality-111` GWT 修正。
- **知识：** [`fill-by-native-id-perf.md`](./knowledge/maps/fill-by-native-id-perf.md) **active**；[`geo-hardcode-recurrence.md`](./knowledge/agent/geo-hardcode-recurrence.md) 第五次教训。

## 2026-09-19 — ADR-071 descope: usable verify blocked (map tokens)

- **Status:** Full-stack descope **implemented** (where2play + places-agent); vitest descope suites green.
- **Usable verify:** **Blocked** — operator cannot run live takeoff/plan/Replan in browser; AMAP + Google Maps API tokens exhausted.
- **DoD:** Usability confirm **pending** until map quota restored.
- **Specs:** [ADR-071](./adr/ADR-071-descope-in-page-refine-replan-only.md) § Verification · [`plan.md`](./plan.md) §7.

## 2026-09-18 — Descope in-page chat refine (ADR-071; replan only)

- **Product:** Remove T9 refine — no post-complete composer, no `/api/chat` refine, no agent `plan_trip` refine.
- **Keep:** Replan dialog + need_input composer during planning.
- **ADR:** [ADR-071](./adr/ADR-071-descope-in-page-refine-replan-only.md); ADR-070 Cancelled.
- **Probe rationale:** [`shanghai-family-refine-probe.md`](./knowledge/agent/shanghai-family-refine-probe.md).

## 2026-09-18 — True-agent refine implemented + Shanghai family probe

- **Agent:** `plan-trip-refine` 主路径改为 `commit_trip.days[]` 完整骨架；去掉 drop 硬闸；`search_places` 回 `names[]`；copy-back 保留 stay kind。
- **BFF:** `changed: false` 追问透传；假成功 reply 仍用 `refine_no_change`。
- **Probe:** [`shanghai-family-refine-probe.md`](./knowledge/agent/shanghai-family-refine-probe.md) — 上海 3 日亲子 / 虹桥亚朵S；「改近一点」追问；「第二天上午不去海洋公园」commit 航海博物馆。Overall PASS。
- **Tests:** `plan-trip-refine.test.ts` · `api-chat.test.ts` 追问透传。

## 2026-09-18 — True-agent refine design approved (ADR-070; specs only)

- **ADR-070:** refine 主路径改为 skeleton LLM + 追问 + 变天 refill；supersede ops-patch + `validateRequiredDropOperations` 主路径；BFF 三态 reply（追问透传，非一律 `refine_no_change`）。
- **Knowledge:** [`plan-trip-refine-t9.md`](./knowledge/agent/plan-trip-refine-t9.md) 重写为 Target 契约与循环。
- **Stories:** `agent-refine-true-agent` · `2play-refine-true-agent`（Approved，待实现）；`chat-01` AC5 标注 Target。
- **Design:** `2play-design.md` §2.4.3 response 三态。
- **代码：** 批准前/本轮 **未改**；下一轮 `plan-trip-refine` + `/api/chat`。

## 2026-09-18 — morning near afternoon + T3 hydrate (`agent-refine-near-keep` / `2play-thread-hydrate`)

- **Agent:** `resolveProximityReplaceSide` — 上午 replace near afternoon; explicit 不去 validation on commit.
- **BFF cache:** skip stale filled day when skeleton attraction names diverge.
- **Client:** hydrate T3 complete line + fillRouteDays on `/api/plan/current`; no retired 4Q greeting after refresh.

## 2026-09-18 — refine thread + indoor (`2play-refine-thread` / `agent-refine-indoor`)

- **Nav:** progress bubble after latest user turn; visible during refine re-fill (`refineInProgress`).
- **Client:** session draft `w2p.chat.draft.session`; no wipe on `planCompleteLine` null or soft replan.
- **Agent:** indoor intent scope (morning / afternoon / whole day); indoor search query templates + replace outdoor attractions.
- **Tests:** DOM order, session draft, Day 3 indoor fixture.

## 2026-09-18 — agent-refine-near (下午 proximity)

- **Agent:** filled schedule + deviations in refine prompt; afternoon/morning index hints; `search_places` with `near` from morning anchor.
- **BFF:** no-op always `play.chat.refine_no_change` (ignore false success agent reply).
- **Tests:** Shanghai Day 2 afternoon proximity; api-chat honesty.

## 2026-09-18 — chat refine hotfix (2play-refine-hotfix / 2play-refine-refill)

- **Root cause:** BFF `mergeRefineSkeletonIntoItinerary` stripped transit/meals on every refine response; client never re-filled.
- **Agent:** `PlanTripResult.changed`; real tool results; substring grounding (`resolveGroundedPlaceName`).
- **BFF:** skip merge; `needs_refill` when changed; `play.chat.refine_no_change` on no-op.
- **Client:** re-fill via `planMode: fill`; in-thread `plan-nav-refine-progress`.
- **Tests:** `api-chat`, `plan-refine-refill`, agent refine unit tests.

## 2026-09-18 — Close MVP-T9 (chat 改行程)

- **agent-chat-93e:** `plan_trip` refine mode + `plan-trip-refine.ts` + unit tests.
- **2play-plan-050:** BFF product LLM removed; `/api/plan` skeleton pipeline only.
- **2play-plan-90e:** `/api/chat` → agent refine; plan-nav composer + localStorage draft; chat-02 E2E un-deferred.
- **Specs:** plan.md / product-backlog → MVP-T10 next; ADR-050 landing note.
- **Knowledge:** `plan-trip-refine-t9.md`, `where2play-t9-close-lessons.md`.

## 2026-09-18 — Close MVP-T8 (usable Confirmed)

- **范围：** places-agent TD-8/9 · where2play T8 UI/E2E · specs。
- **关门：** MVP-T8 标 **Done**；`plan.md` / `product-backlog.md` 下一步 **MVP-T9**；用户 usable Confirmed。
- **工程：** agent `tsc` 绿（`PlanTripResult.itinerary` 含 `fillReachedTripComplete`）；E2E mvp1/mvp2/mvp3/mvp10-structure/mvp-t3/mvp10-live 绿；chat02 defer T9。
- **知识：** [`where2play-t8-close-lessons.md`](./knowledge/web-app-development/where2play-t8-close-lessons.md)。
- **已知 open：** 探针 test12 台北；chat-02 E2E → T9。

## 2026-09-18 — T8 UI hint placement + E2E drift fixes

- **范围：** where2play `plan-assistant-nav` · `plan-page` · E2E · unit tests。
- **UI：** `assistant_next_hint` 与 soft replan chip 仅在 `planCompleteLine` 之后渲染（不再在 skeleton ready 时出现在 Day 1 前）。
- **E2E：** 新增 `e2e/takeoff_helpers.py`（takeoff-11 + confirm）；`test_mvp10_structure` / `test_mvp3_live` 对齐 T3；`test_chat02` defer MVP-T9（SKIP exit 0）；mvp1 链全绿。
- **验证：** `plan-t3-ui-fixes` + `plan-assistant-t3-thread` vitest；`make test-e2e-mvp1` · `mvp3-live` · `mvp10-structure` · `chat02`。

## 2026-09-18 — Reslice remaining MVP into T8/T9/T10 (3 batches)

- **范围：** `plan.md` · `product-backlog.md` · `T5-plan.md` · `change-log.md`。
- **变更：** 用户三批拆分 — **MVP-T8** 行程规划闭环（103/104 + T5 TD-8/9/10 + 93f/90f + 探针/e2e）；**MVP-T9** chat 改行程（93e/050/90e）；**MVP-T10** 出行贴士+保存（93d/90d + saved/replan/PDF）。T5 TD-3–TD-7 标 Done；剩余并入 T8。
- **变更（2026-09-18 MVP-T8 实现）：** TD-8/9 `fill-trip-status` + `applyFillTripStatusGate`；TD-10 transit grid + phase meta「框架」；103/104 Done；93f/90f 故事签收；回归探针/e2e 进行中。
- **关联：** 无新 ADR；T9/T10 待 T8 Done 后详细计划。

## 2026-09-18 — Sync plan + product-backlog MVP status

- **范围：** `plan.md` · `product-backlog.md` · `T5-plan.md` · `agent-stories.md` · `agent-design.md`。
- **变更：** `as_of` 2026-09-18；T3++ agent `110a`–`110d` 标 **Done(producer)**；T4 **Cancelled**（ADR-069）；MVP-T5 **In progress**（TD-3–TD-7 Done，**TD-8 next**）；`2play-plan-100` Done；质量债 111/112 Done；12-case 探针 11/12 ready 记入 §0。
- **关联：** [`T5-plan.md`](./T5-plan.md) §9；无新 ADR。

## 2026-09-18 — Close `agent-test-112` suite hygiene

- **范围：** `plan-next-stop.test.ts` `place()` 夹具；`make-itinerary.test.ts` ADR-069/068/067 对齐。
- **变更：** cluster dwell 夹具 `native_id` 改为 `g-${name}`（避免 fill dedup 误删邻居 B）；删 must_see 标注/排序与 registry merge 用例；pace 软节奏用例改为允许 1 站/日、仅极端超量裁剪至 6；季节 prompt 断言改为无「不要因季节硬删」硬规则。
- **验证：** `tsc --noEmit`；plan-next-stop 47/47、make-itinerary 66/66、eligible/stay-photo/guard 155 全绿。
- **关联：** ADR-069、ADR-068、ADR-067；下一步 **MVP-T5 TD-8**。

## 2026-09-17 — Close `agent-quality-111`; next `agent-test-112`

- **完工：** `agent-quality-111` — 删城市 POI 正则与圣名 cognate、vendor search 别名、`tests/no-city-hardcode.test.ts`。DoD：usable Confirmed 2026-09-17。
- **下一故事：** `agent-test-112` — (1) `plan-next-stop.test.ts` cluster dwell A/B 夹具 unique `native_id`（现共用 `g1` → 期望 20 实得 45）；(2) `make-itinerary.test.ts` 对齐 ADR-069（删 `[must-see]` / 硬节奏裁剪等过时断言）。
- **关联：** ADR-042 Update 2026-09-17；ADR-069；T5 仍待 TD-8，排在 112 之后。

## 2026-09-17 — Remove city POI regex, proper-name cognates, add ADR-042 CI guard

- **范围：** `eligible-attraction.ts`、`plan-next-stop.ts` `matchCardByPointer`、骨架 prompt、`tests/no-city-hardcode.test.ts`。
- **Item 1：** 删除 `isVagueAreaName` 中田子坊/城隍庙/银座等城市 POI 白名单；只保留通用后缀（District/街区/古镇/商城…）。
- **Item 2：** 删除 `PROPER_TOKEN_COGNATE_GROUPS`（jorge↔george 等）。`matchCardByPointer` 为 native_id → 精确名 → 去音调子串（长度≥4）→ miss 后 vendor search。骨架 prompt 要求 verbatim 复制候选名、禁止翻译地名。venue-type 词表去重；季节 ontology 去掉西湖十景专名示例与秋叶原意图词。
- **Item 3：** `tests/no-city-hardcode.test.ts` 扫描 `src/core` + `src/mcp`（排除 `*.test.ts`）。
- **验证：** `npx tsc --noEmit`；eligible / stay-photo / no-city-hardcode 全绿；`make-itinerary` verbatim prompt 用例通过。`plan-next-stop` 集群 dwell `A`/`B` 单测仍期望 20/35 实得 45（卡片共用 native_id `g1`，与本次无关的既有夹具问题）。Lisbon 4d probe `test4` **ready**（68s，pool=17 Google）。
- **关联：** ADR-042；无新 ADR。

## 2026-09-17 — Live 12-case skeleton probe + 2play T3/MVP-2 E2E

- **范围：** `probe-prompt-test-cases-skeleton.ts`（expand_radius 二段 affirm）；where2play T3 Hangzhou 骨架 E2E；`make test-e2e-mvp2-live`。
- **探针（live + probe cache）：** 11/12 `ready`。test2 杭州 **未**误触发 expand_radius；test10 香港 dual AMAP+GOOGLE 且 `ready`；test4 里斯本 4d `ready`（未问 expand，Belém/Jerónimos 在骨架）。**test12 台北失败：** pool=3 Google、骨架校验（西門紅樓复用 / day2 0 attraction）。
- **探针脚本：** `need_input.expand_radius` 时用同一 `trip_id` + `answers.expand_radius=yes` 再调 `planTrip`。
- **2play：** `e2e/e2e_t3_skeleton_hz.py` + Makefile `test-e2e-mvp-t3` 通过（杭州 3d 骨架 18.5s，截图 `signoff-hangzhou-3d-t3-ui.png`）。`test_mvp2_live.py` 对齐 takeoff-11：杭州 3d 保存→详情→取消收藏通过。`plan-page` 在 T3 `done` 后不再把 `generating` 钉死（否则保存按钮永禁）。助手侧栏会挡住保存点击，E2E 先关 `plan-nav-close` 再 force click。London 1d 在 45s 仍停在 filling，未作为本次保存闭环目的地。
- **验证：** 探针 JSON `places-agent/tmp/probe-prompt-test-cases-skeleton.json`；T3 截图；`python3 e2e/test_mvp2_live.py` ok。
- **关联：** ADR-057；`prompt-test-case.md` TC-PROBE-12 Partial；无新 ADR。

## 2026-09-17 — Dedup pool alias to places-agent + typecheck cleanup

- **范围：** where2play `lookupPoolCandidate`；places-agent / where2play tsc 错误。
- **跨包重复：** where2play 曾镜像 agent 的 cognate / venue-type 模糊匹配（`poolAliasMatches`）。ADR-010 下两仓独立 remote，不建共享包；匹配逻辑归 places-agent（`sharedProperToken` + `matchCardByPointer`）。where2play 仅保留 native_id + exact / 去音调 / 子串薄缓存。真正 `@places/core` 共享包需修订 ADR-010 + workspace，留作后续 ADR。
- **变更：**
  - 删除 where2play cognate 表与 Castelo↔Saint George UI 用例；Belém 用例改为同词序去音调（Torre de Belém ↔ Torre de Belem）。
  - places-agent：probe `result.itinerary?.skeleton`；去掉 `must_see` 类型访问；fill mock 补 `transit_outcome` / `notes`。
  - where2play：`PlanTripData` 补 `phases` / `itinerary`；`AgentNeedId` 加 `expand_radius`（T3-only，不进 intake b–h 映射）。
- **验证：** 两仓 `tsc --noEmit`；相关 vitest。
- **关联：** ADR-010；ADR-051；ADR-069；无新 ADR。

## 2026-09-17 — Hangzhou false expand_radius (ATTRACTION_ALLOW vs eligible)

- **范围：** places-agent `plan_trip` 110d 薄池计数 `countGroundedAttractions` / `intakeEligible`。
- **现象：** 杭州 3 天误提示「附近景点偏少…扩大至约 160 公里」。
- **根因：** 扩圈闸用 `isAttractionish`→`filterAttractionPlaces`（ATTRACTION_ALLOW）。高德卡常无 `category`，西湖名（苏堤 / 灵隐寺 / 雷峰塔景区）不匹配 allow，且 `景区` 还在 VISIT_DENY → 本地景点被计成接近 0，虽 nominate 已按 `isEligibleAttraction` 入库。
- **变更：** 薄池计数与 intake 过滤改为 `filterEligibleAttractions`（与 discover 落池一致）；去掉重复的 `isAttractionish`。
- **验证：** 新增杭州无 category 卡不触发 expand；110d 原薄池用例仍绿。
- **关联：** ADR-067 / `agent-discover-110d`；无新 ADR。

## 2026-09-17 — Castelo de São Jorge list thumbs (PT↔EN + Garden false match)

- **范围：** places-agent `sharedProperToken` / `plan_next_stop` 填站缩略图；where2play `lookupPoolCandidate`。
- **核实：** 与 Belém 同类——葡语站名 / 英语池名对不上。`Castelo de São Jorge`↔`Saint George Castle` 原先 `sharedProperToken` 为 false（`jorge`≠`george`）；唯一模糊命中还会绑到 `Garden of the Castle of São Jorge`（同 patron、无图），挡住 search 回退；即使池内已 resolve 图，`displayCurrentStop` 仍按骨架英语名精确匹配 → `card` 为空。
- **变更：** 专名 token 去音调（Belém↔Belem）；圣名 cognate（jorge↔george 等，非城图表）；主场馆类型一致才模糊匹配（castle≠garden）；enrich 后把 `native_id` 打到 stop，保证 display 拿到已解析卡。UI 池查找仅 exact/去音调（cognate 归 agent，见同日 Dedup 条目）。
- **验证：** eligible-attraction + plan-next-stop-stay-photo + itinerary-skeleton-map 单测；live fill E/F/G/H（Garden-only / Saint George / Belem 无音调）均有 https 图。
- **关联：** ADR-051；2026-09-14 Belém/Jerónimos 条目；无新 ADR。

## 2026-09-17 — Fix 6 e2e probe quality issues (P1–P6)

- **范围：** where2play fill stream 超时/异常关闭；`plan-skeleton-fill` patch 重试上限；places-agent 酒店名泄漏骨架、nominate 薄池重试、Google discover `bias_radius_m=50km`。
- **根因（探针 12 case）：** fill 无 AbortController → UI 排队挂起；HK 酒店名被当景点；成都 nominate 偶发极薄；台北 Google 默认 5km bias 过窄。
- **变更：**
  - P1/P2：`runFillFromSkeleton` / `runPlan` 300s abort + `authNdjsonEvents` reader cancel；stream 无 done/error → `play.plan.assistant_fill_timeout`（EN/CN/HK/TW）。
  - P3：`skeleton_patched` 同 stop 最多 3 次后强制前进。
  - P4：骨架 prompt + overlay 明确 origin 仅为 stay；`dropUnknownAttractionStops` 丢弃与 stay 同名的 attraction。
  - P5：nominate 少于 8 名时再试一次（至少 15 名指令）。
  - P6：discover / ground / must_include 的 `searchPlaces` 传 `bias_radius_m: 50_000`。
- **验证：** `make-itinerary` hotel-as-attraction + CJK stay variant 单测；i18n catalog；探针 test10 香港 `ready`（换无景点词酒店名 + stay 繁简匹配）；test6/12 薄池正确走 `expand_radius` need_input。
- **关联：** ADR-057；ADR-067；无新 ADR。

## 2026-09-17 — Expand prompt-test-case.md to 12 cases

- **范围：** `specs/agent-specs/prompt-test-case.md`（5→12 case）；`places-agent/scripts/probe-prompt-test-cases-skeleton.ts`（CASES 数组同步）。
- **目的：** 强化 e2e 探针覆盖——供应商路由（AMAP/Google/HK 双源）、POI 密度边界（expand-radius）、跨语言名匹配、locale（CN/EN）、无起点、多日远集群。
- **变更：** test4 里斯本 3d→4d 触发 expand-radius；新增 test6 成都美食、test7 北京家庭度假 5d、test8 厦门朋友无起点、test9 深圳商务、test10 香港亲子双路由、test11 曼谷 4d EN、test12 台北 TW 措辞。7 AMAP（免费）+ 5 Google（opt-in + 缓存）。
- **验证：** 待跑 `probe-prompt-test-cases-skeleton.ts`。
- **关联：** ADR-057 成本策略；ADR-067 todo6b expand-radius；无新 ADR。

## 2026-09-14 — Lisbon Belém / Jerónimos list thumbs missing (Google)

- **范围：** places-agent `plan_next_stop` 景点图 enrich；`discover_places` 提名/must_include 补图；`attachNativeIds` / `matchCardByPointer` 别名。
- **根因：** 骨架用葡语名（`Torre de Belém` / `Mosteiro dos Jerónimos`），候选池是 Google 英语名（`Belém Tower` / `Jerónimos Monastery`）→ 精确名匹配失败 → `stop_display.card` 为空无图。Google 本身有图（media / details 正常）。
- **变更：** 池未命中时按站名 search + resolve 图；别名唯一匹配；多 Belém 时优先有图卡；discover 对提名/must_include 再跑 `resolveDisplayPhotosForCards`。
- **验证：** live `plan_next_stop` PT↔EN；`plan-next-stop-stay-photo` locale alias 用例。
- **关联：** ADR-051；无新 ADR。

## 2026-09-14 — Lisbon 4d fill stops after day 2

- **范围：** where2play `plan-skeleton-fill` 瞬时重试；`plan-page` / `plan-itinerary-view` 填站失败后解锁后续日 tab。
- **根因：** 填站中途 agent（tsx watch）因文件保存重启 → `errors.provider_failed` 整趟中止，只留下已填的 Day1–2；T3 `generating` 在 done 后仍为 true，未填日 tab 被 queued 锁死。
- **变更：** `plan_next_stop` 对 `provider_failed` / `arrange_timeout` 最多再试 2 次（`PLAN_NEXT_STOP_RETRY_MS`）；填站失败后 `planSubPhase=idle`；`queueFutureDays` 仅在 filling 时锁定后续日。
- **验证：** `plan-skeleton-fill` transient retry 用例。
- **关联：** 无新 ADR。

## 2026-09-14 — Place sheet title flash Latin→CJK

- **范围：** `preferSlotDisplayName` / address；ADR-052 D9 文案；2play AC43/AC19。
- **根因：** D9 只防 CJK 被拉丁覆盖；CN locale 详情返回「国家瓷砖博物馆」会盖掉列表 `Museu Nacional do Azulejo`。
- **变更：** 跨文脚本（CJK↔拉丁）一律保留槽位名/地址；同文脚本仍可用详情 refinement。
- **验证：** `place-display-prefer` 测试。
- **关联：** ADR-052 D9 对称澄清；无新 ADR。

## 2026-09-14 — Fill list missing AMAP thumbs (details has photo)

- **范围：** places-agent `slimArrangeCandidate` / `resolveDisplayPhoto` / attraction fill enrich；where2play pool name/paren lookup。
- **根因：** slim 用 `isDisplayablePhotoUrl` 未先 http→https，丢掉高德 CDN 图；`resolveDisplayPhoto` 只对 Google 调 details，景点 fill 不补图。
- **变更：** slim 用 `pickDisplayablePhotoUrl`；AMAP native_id 也可 getDetails 补图；非 stay fill 缺图时 enrich；池名匹配全角括号。
- **验证：** resolve-display-photo + itinerary-planner slim 用例。
- **注：** 逍遥轩 AMap 官方无 photos 字段，详情也无法从供应商出图（与茶叶博物馆不同）。
- **关联：** ADR-051；无新 ADR。

## 2026-09-14 — AMap 龙井村 tip id: no details / no photo

- **范围：** places-agent `hydrateNominatedCard`；where2play place sheet `/api/places` name fallback；AMAP native_id 允许 `BV…`。
- **根因：** inputtips「龙井村」有坐标但 `/place/detail` 空 pois、无图；hydrate 因有坐标跳过 search；sheet 调详情失败且列表无缩略图。
- **变更：** 无 https 图的 tip 强制 search 升级；详情失败时用 `name`(+city/near) 搜可展示卡；AMAP id 门控含 `BV`。
- **验证：** `ground-nominated-name`、`place-details-resolve`、`place-native-id`。
- **关联：** ADR-052；无新 ADR。

## 2026-09-14 — Origin stay missing photo

- **范围：** places-agent `resolveOriginStay` providers + photo name fallback；where2play resolve-origin / plan-page `originStay`。
- **根因：** intake 命中只存店名；API 不回传 `photos`；Google 搜索卡常无 https 图；`resolveOriginStay` 未传 `providers`。
- **变更：** `ensureOriginHitPhotos`（agent details）；API 返回 photos；UI 持久化 `originStay` 并传 fill；agent 传 providers；media 路径可解析为 photo name。
- **验证：** `plan-resolve-origin` 25/25。
- **关联：** ADR-051 / ADR-053；无新 ADR。

## 2026-09-14 — Place sheet: reject verify_* native ids + soft degrade

- **范围：** places-agent `attachNativeIdsToSkeleton`；where2play `place-native-id`、slot 映射、place sheet、`/api/places`。
- **根因：** 部分站带了非厂商 id（如 `verify_belem`，POC/幻觉指针）→ `get_place_details` 502 → 「无法加载地点详情」且常无图；假 id 还挡住池内真实 `ChIJ…`。
- **变更：** `isResolvablePlaceNativeId`；骨架附着时覆盖 harness id；slot 优先池内可解析 id；sheet 对不可解析/失败软降级（保留 slot 图文）。
- **验证：** where2play place-native-id + skeleton-map；agent overwrite_harness 用例。
- **关联：** ADR-051；无新 ADR。

## 2026-09-14 — Fill photos + Day-2 lodging re-search

- **范围：** places-agent `dispatch` filled 写；where2play `plan-skeleton-fill` / `itinerary-skeleton-map`。
- **根因（缺图）：** TD-6 SoT 读 `filled`，但 `plan_next_stop` 只写了 bare `next_stop` 指针，丢掉 `stop_display.card.photos`。
- **根因（里斯本 Day 2 慢/卡住）：** 每日 `origin_mode` stay 不在候选池 → 无 `native_id`/图 → 反复 lodging 搜索（单次可达 ~100s）。
- **变更：** filled 写入完整 `stop_display.stop`；BFF coalesce 信封图；fill 注入 `constraints.originStay` 并 stamp 指针。
- **验证：** where2play skeleton-map + fill 测试 27/27。
- **关联：** ADR-051 / ADR-053；无新 ADR。

## 2026-09-14 — MVP-T5 TD-7 + fill invalid_input (stay-less day)

- **范围：** where2play `plan-skeleton-fill` + `plan-page`；`T5-plan` TD-7/U3；`2play-design` §4.11。
- **根因（`errors.invalid_input`）：** hotel skip / 无 stay 骨架时首站 `plan_next_stop` 既无 `origin_mode` 也无 `current_stop` → agent Zod 400。
- **变更：** fill 用 trip `dayOrigin` 种子 `current_stop`；fill 期间锁 Day 1（不跟 fill `dayIndex`）；`liveSlots` 仅追加 Day 1。
- **验证：** `plan-skeleton-fill` 新增 stay-less 用例（18/18 pass）。
- **关联：** TD-7 Done；无新 ADR。

## 2026-09-14 — MVP-T5 TD-6: progressive fill SoT from fetch filled

- **范围：** where2play BFF `plan-skeleton-fill`；`2play-design` §4.11 U2。
- **变更：** `stop_filled` / transit 映射自 `fetch_trip_details(filled,cursor)`（`latestFilledStopFromSlice` + `mapFilledStopToDisplay`）；信封仅降级；客户端仍 NDJSON 追加 `liveSlots`。
- **验证：** `plan-skeleton-fill` TD-6 用例（错误信封 vs 正确 filled）；`plan-fetch-trip` unwrap。Day-1 tab 锁属 TD-7。
- **关联：** T5-plan TD-6；无新 ADR。

## 2026-09-14 — MVP-T5 TD-5: tokyo origin stay cross-script pick

- **范围：** `places-agent` `pickLodgingStayCard` + full-loop `resolve_origin_stay`；探针 tokyo。
- **根因：** EN 酒店名 vs Google CN 标题导致 pick 失败 → agent 路径无 name-only 回退 → 重试烧尽 → 502 `provider_failed`（非区域路由错误）。
- **变更：** 唯一 lodging hit 接受；agent/legacy 共用 `nameOnlyOriginStay`（GOOGLE_MAPS 默认）；`resolve_origin_stay` once-guard。
- **验证：** 单测 TD-5 + sole_lodging；live tokyo **ready 20/20 (100%)**。无新 ADR。

## 2026-09-14 — Decision: TD-2a/TD-2b re-eval after probes + TD-4

- **范围：** MVP-T5 / `T5-plan.md` §5.4 · §4.4 · §9。
- **变更：** TD-2a **维持 soft-locked**（本批 fill/餐/硬闸无新 LLM；非永久 ADR，升格需用户明示）；TD-2b **维持 rejected**（满填后质量债走 TD-8/TD-9）。下一步开 **TD-5**（tokyo provider）。
- **关联：** 探针轮次 0–2 + TD-4 xian skip；无新 ADR。

## 2026-09-14 — MVP-T5 TD-4: HTTP answers.hotel resume

- **范围：** `places-agent` `plan_trip` HTTP answers；`where2play` BFF body 转发；探针 xian。
- **变更：** `answers.hotel`（店名 → origin；skip → 骨架无起点）；dispatch 转发 hotel+expand_radius；`resolveHotelAnswer`；expand_radius 回归保持。
- **关联：** `T5-plan.md` TD-4；无新 ADR。

## 2026-09-14 — Decision: TD-2b reject day-review LLM after fill

- **范围：** MVP-T5 / `T5-plan.md` TD-2b · TD-11。
- **变更：** 用户同意——**不做**代码 fill 后的 LLM 当日 review。依据：`prompt-test-case` ready 子集 fill 100%；残留回弹/末时/transit 归 S6/S7；西安 hotel / 东京 provider 归 S2/S3。S6 后若仍系统性差，再议只读 review→deviations（须新故事+ADR）。
- **关联：** 探针轮次 2；无新 ADR。

## 2026-09-14 — MVP-T5 S1: full-loop fill early-stop fix (A+B)

- **范围：** `places-agent` `plan_trip` 全环（`skeleton_only=false`）；`specs/T5-plan.md` S1 / TD-3。
- **变更：** 收紧 `stop` 工具描述（`FULL_LOOP_STOP_TOOL_DESCRIPTION`）+ `buildFullLoopSystemPrompt`（须 `trip_complete` 后再 commit/stop）；单测 `MVP-T5 S1 A+B`；探针脚本 `scripts/probe-t5-fill-review.ts`。
- **验证：** 上海 / 杭州 / 里斯本 fill **100%**（此前 ~17–20% 早停）。无新 ADR（提示词硬化；见 [`knowledge/agent/full-loop-early-stop-ab.md`](./knowledge/agent/full-loop-early-stop-ab.md)）。
- **非范围：** day-review LLM、S2 answers、2play UI。

## 2026-09-14 — Policy: no agent prompt must-see features + docs sync

- **范围：** `plan.md` / `product-backlog.md` / ADR-069；纠正 plan §3d 过时「110a AC Ready」勾选。
- **变更：** 产品政策再确认——**不再做 agent 提示必去点相关功能**（不恢复 C/D、不新开必去标记/理由/必去 UI）；保留 A（发现）与 B（must_include）；`agent-iconic-69` → Cancelled；`110f` 改称「知名景点覆盖 monitor」（非必去产品）。
- **关联：** [ADR-069](./adr/ADR-069-delete-must-see-marking-and-t4-reasons.md) amendment；下一步仍为 **MVP-T5**。

## 2026-09-14 — Prompt: senior itinerary planning expert persona (skeleton + planner overlays)

- **范围：** `places-agent` 骨架/规划 overlay 人设行；决策「overlay 保持英文、全 locale 共用」（维持现状）。
- **变更：** `itinerary-skeleton.md` / `itinerary-planner.md` 人设从「knowledgeable local guide」升级为「senior itinerary planning expert」+ goal directive（按起飞约束设计最符合需求的骨架）+ delegation boundary（密度/节奏/动线归 LLM，宿主只守结构安全闸）+ implicit preference 推理指令；保留 local guide 作为排序视角。中文对照文本仅存档于对话，不落 locale 化 overlay 文件。
- **去硬编码（方案 A）：** 删除人设中三条具体映射（亲子→乐园全天 / 远簇→独立一日游 / relaxed→少排站），改为「Infer implicit preferences from the constraints using the glossary below; do not pad days to meet a count」——推理框架归人设，具体约束效果归 11 项 glossary（ADR-068），避免预判工作流。
- **验证：** 方案 A 后 `prompt-test-case.md` 5/5 ready（上海：海昌 day1 + 迪士尼 day3 收官；杭州 relaxed 3–4 站；西安兵马俑远郊日；里斯本 Cascais 一日游；东京动漫动线递进）；`itinerary-skeleton-overlay.test.ts` 4/4。
- **关联：** ADR-068（软节奏 vs 硬安全闸的人设实现细节）；ADR-069（must_see 删除后以软偏好+人设替代 C 的收尾）。执行中发现工作区 `itinerary-skeleton.md` 曾被回退到 110e 前旧版（硬下限回归），已从 HEAD 恢复后再套人设。

## 2026-09-11 — ADR-069 delete must_see marking + close MVP-T3++Q

- **范围：** 删 must_see 标记层（C）与 T4 必去理由交互（D）；交付 `110g`→`110b`→`110c`→`2play-plan-103`→`110d`→`2play-plan-104`。
- **变更：** 保留发现提名（A）与 must_include 硬闸（B）；骨架 deviations 透明展示；POI 不足 `expand_radius` need_input；探针 5/5 支持删 C。
- **关联：** [ADR-069](./adr/ADR-069-delete-must-see-marking-and-t4-reasons.md)；知识 [`knowledge/agent/must-see-marking-deletion-probe.md`](./knowledge/agent/must-see-marking-deletion-probe.md)；`agent-discover-110g`/`110b`/`110c`/`110d` · `2play-plan-103`/`104`。

## 2026-09-11 — MVP-T3 / T3+ usable Confirmed + ADR-067 plan reschedule

- **范围：** T3（`2play-plan-101` · `agent-itinerary-100`）+ T3+（`102`–`109`）标 Done；计划重排插入 MVP-T3++（`110a`–`110d` · `2play-plan-103`/`104`）于 T4 之前。
- **变更：** usable Confirmed 2026-09-11（计划实现批准）；as-built 模板池质量债移交 ADR-067；T4 依赖改为 T3++；AC stubs + test-plan §44 / §14。
- **关联：** [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)；知识 [`knowledge/agent/t3-template-pool-debt-to-adr067.md`](./knowledge/agent/t3-template-pool-debt-to-adr067.md)。

## 2026-09-11 — Decision: ADR-067 LLM-driven discovery replaces stops-pool

- **范围：** 6 项诊断决议汇总（todo1-6），方案 A 落地方向。
- **变更：** 决定用 LLM 驱动发现取代代码模板 stops-pool（方案 A）；季节规则删骨架段（2a-none）；发现提示词用 OptA 单条消息+三补丁；trip_type 全去枚举（a3）；other 去"偏好"二字（b2）；远簇拆分改校验不修补+透明展示；北大壶 POI 不足用 deviation+用户确认扩搜索半径。
- **关联：** [ADR-067](./adr/ADR-067-llm-driven-discovery-replaces-stops-pool.md)（supersede ADR-062/065）；探针 `scripts/probe-discover-style-ab.ts` + `tmp/probe-discover-style-ab.json`；`agent-discover-110`（next batch）。

## 2026-09-11 — Impl: agent-itinerary-109 full Takeoff-11 prompt intake

- **范围：** `formatTripPrefsForPrompt` ISO 日期；`BUDGET_HINTS` mid/comfort/economy/luxury；start_time/transit 软偏好句；`makeItinerary` 透传 budget。
- **变更：** 起飞 11 项在骨架 prompt 中完整、无歧义；骨架 JSON 仍禁止 times/transit。
- **关联：** TC-T3-109-* / agent-itinerary-102。

## 2026-09-10 — Impl: agent-itinerary-108 eligibility leakage

- **范围：** `ATTRACTION_FRAGMENT_DENY` 停车点/充电站/公交站；`isEligibilityNoiseCategory`（交通设施/汽车服务，不含娱乐场所）；`isEligibleAttraction` 调用。
- **变更：** 主题公园停车点、充电站、公交站出池；乐园+娱乐场所仍 eligible（无 105 回退）。
- **关联：** ADR-066 / ADR-042 / TC-T3-108-*；知识 [`knowledge/agent/eligibility-leakage-transit-auto.md`](./knowledge/agent/eligibility-leakage-transit-auto.md)。

## 2026-09-10 — Impl: agent-itinerary-107 kids prompt rank + overlay

- **范围：** `formatTripPrefsForPrompt` kids 池内 park/乐园 排序；overlay 去品牌 forbid。
- **变更：** 软偏好仅限候选池；不要求发明品牌 POI。
- **关联：** ADR-066 / TC-T3-107-*。

## 2026-09-10 — Impl: agent-itinerary-106 kids theme-park queries

- **范围：** `skeletonPoolQueries` CN 主题公园优先；EN zoo/aquarium under cap。
- **变更：** family_kids 含主题公园；couple/历史不加游乐园模板。
- **关联：** ADR-066 / TC-T3-106-*。

## 2026-09-10 — Impl: agent-itinerary-105 theme-park eligibility

- **范围：** `place-filters` ATTRACTION_ALLOW + resort lodging narrow；`isAttractionish` 乐园模板。
- **变更：** Resort+tourist_attraction / …乐园 / 欢乐谷 可入池；真酒店仍 lodging；无城市→POI 表。
- **关联：** ADR-066 / TC-T3-105-*。

## 2026-09-10 — Specs: theme-park pool D（105–107 / ADR-066）

- **范围：** agent-design 建骨架；ADR-066；`agent-itinerary-105`–`107`；test-plan §42；backlog / plan。
- **变更：** 资格闸乐园/resort → 亲子主题公园 query → 池内排序提示；无城市百科、不发明池外名。
- **关联：** ADR-042/065/066。

## 2026-09-10 — Impl: agent-itinerary-104 geo far-cluster days

- **范围：** `ensureFarClustersOwnDays`（haversine 簇阈值）；`makeItinerary` 在 trim 后调用；`must_include` 空仍生效。
- **变更：** 市中心与远郊混日拆成独立 day_theme；不发明未排程 POI；无城市百科。
- **关联：** ADR-065 C / TC-T3-104-*。

## 2026-09-10 — Impl: agent-itinerary-103 preference pool queries

- **范围：** `skeletonPoolQueries`（基线 museum/landmark + cap 偏好模板）；`expandPlacesForSkeleton` 使用起飞 trip_type/other。
- **变更：** family_kids/儿童 → 亲子/游乐园类 query；情侣不加游乐园；无城市 POI 表（ADR-042）。
- **关联：** ADR-065 / TC-T3-103-*。

## 2026-09-10 — Impl: agent-itinerary-102 skeleton prompt prefs/season

- **范围：** `formatTripPrefsForPrompt`；`buildSkeletonUserMessage` 旅人块；`plan_trip` 停 slug soup；overlay 季节/偏好；BFF `bounds.end`。
- **变更：** 起飞字段以显示名+season rule 入骨架提示；`other` 为偏好非 must_include。
- **关联：** ADR-065 / ADR-042 / TC-T3-102-* / TC-T3-BFF-01。

## 2026-09-10 — Specs: skeleton quality 102/103/104（ADR-065）

- **范围：** `agent-design` 建骨架；ADR-065 follow-up；`agent-itinerary-102`–`104` AC；`agent-test-plan` §41；backlog / `plan.md`。
- **变更：** T3+ 质量拆为提示 → 偏好池 → geo 闸三故事；不重开提名、无城市百科。
- **关联：** ADR-042/062/065。

## 2026-09-10 — Bugfix: takeoff trip-type presets clipped + Xi'an skeleton hang UX

- **范围：** takeoff `PlanCombo` / `mockup-travor.css`；`plan-t3-hydrate` progress；`auth-api` abort；`plan-page` T3 130s ceiling；agent `plan_trip` skeleton_only 允许无起点 + stay 落空 name-only。
- **变更：** 起飞行程类型恢复 5 项预设（情侣浪漫 / 家庭度假 / 亲子玩乐 / 吃喝打卡 / 都市漫步）且可手输；combo `overflow:hidden` 打开时改为可见。助手失败后不再卡在「正在生成框架」；无酒店或住宿搜不到时仍可出骨架。
- **关联：** `2play-plan-100` / `2play-plan-101`、ADR-062/063。

## 2026-09-10 — Bugfix: T3 constraints missing hotel/departure after takeoff

- **范围：** `plan-page.tsx` `beginT3Progress`；`plan-page-t3.test.tsx` AC1。
- **变更：** 起飞提交后把 origin / startTime / other 写入 `intakeAnswers`，出行限制 panel 显示每日起点与出发时间（不再「无酒店」/默认掩盖用户值）。
- **关联：** `2play-plan-101` AC1、ADR-062。

## 2026-09-10 — agent-design：能力逻辑与提示组合 + ADR-065 提名 vs 骨架（B+C Accepted）

- **范围：** `agent-design.md` 新增「能力逻辑与提示组合（内部意图）」节，定义收边界 / 必去提名 / 建骨架 / 填细节 / 四卡五项内部意图的触发、输入、逻辑、提示组合、事实闸；评估提名 vs 骨架关系选项；[ADR-065](./adr/ADR-065-nominate-vs-skeleton-relationship.md) Accepted。
- **变更：** 按 TRUE AGENT 原则（环持有 + 事实闸由代码 + 每项能力独立 act-or-stop）采纳 **B+C**（提名独立能力产 must-see+理由入池，骨架当偏好消费；外加代码侧 geo 多样性闸让 T3 `skeleton_only` 远簇成日，不重开提名、无城市 CATALOG）。
- **关联：** ADR-042/050/060/062/063/065、`make-itinerary.ts` `buildSkeletonUserMessage`、`itinerary-planner.ts` `nominateMustSeeViaLlm`、`agent-itinerary-101` / `2play-plan-102`（T4）、T3 geo 多样性闸 story（新）。

## 2026-09-10 — Assistant T3 UX: notice bubbles + progress + tripType display

- **范围：** `mockup-travor.css` `.bubble--agent-notice`；`plan-assistant-nav` T3 progress bar + rail 55%；`beginT3Progress` 乐观 phases；`formatTripTypeDisplay` 框架就绪句。
- **变更：** 助手说明恢复聊天气泡；生成中可见步骤态 + 进度条/文案；就绪句显示「情侣浪漫」而非 `couple_romance`。
- **关联：** ADR-062、`2play-design` §4.7.1 A2/A3、`2play-plan-101`。

## 2026-09-10 — ADR-064 Option A + takeoff dest/confirm bugfix

- **范围：** ADR-064 Accepted（Option A）；`2play-design` §4.7 confirm i18n；`2play-plan-100` US6；mock 单确认层；AMAP admin 缺省 country + 市 strip；起飞确认管道（where2play）。
- **变更：** Enter/CTA → 确认摘要层；起点歧义先起点层；`里斯本`/`杭州` 验真标签；四 locale `submit_confirm_*`；TC-T2-100-10…14。
- **关联：** ADR-061/064、`2play-plan-100`、`agent-geocode-100` AC4。

## 2026-09-10 — ADR-064：起飞提交确认 · 起点先于确认（未实现）

- **范围：** ADR-064；mock `06-plan-takeoff-11.html`（`confirm_a` / `confirm_b` / `origin_then_confirm`）；`2play-design` §4.7 指针。
- **变更：** 记录管道决策（Enter+CTA → 确认；起点歧义只开起点层，再开确认）；确认体 A/B 待产品用 mock 选定；**不实现应用代码**。
- **关联：** ADR-061、起飞 takeoff bugs 修复待确认后开工。

## 2026-09-10 — MVP-T3 实现：skeleton_only + 助手进度 + 框架 UI

- **范围：** places-agent `plan_trip`（`skeleton_only`、phases、无四问/无 fill）；where2play submit→progress→hydrate；BFF trip + fetch；i18n 四 locale；E2E `test_mvp_t3.py`；ADR-063。
- **变更：** Takeoff 提交后助手接管与 `.plan-progress`；主区框架 slots；MCP 全环调用方省略标志行为不变。
- **关联：** ADR-062/063、`2play-plan-101`、`agent-itinerary-100`。

## 2026-09-10 — MVP-T3 Story 0：artifact cleanup

- **范围：** `specs/archive/`（`mvp-1t-closing-plan.md`、`agent-plus-2play-chat.md`、`real-agent-refactory-chat.md`）；`tmp-0909.md` stub；链接改指 `agent-design` / `plan.md` / ADR-061/062。
- **变更：** 聊天 dump 与收尾台账移出 SoT；Story 0 / TC-T3-101-00 Done。
- **关联：** ADR-062、`2play-plan-101` AC0。

## 2026-09-10 — MVP-T3 设计：按屏 UI + 技术合同 + 时序图

- **范围：** `2play-design` §4.7.1（A 按屏 / B BFF·phase / C mermaid）；`agent-design` MVP-T3（入参归一、停止策略、phase 合同、fetch fields、agent 时序图）。
- **变更：** 锁定 2play↔助手↔places-agent↔Trip/vendors 数据流；T3 停在 skeleton；phase→i18n；真源 fetch。
- **关联：** ADR-062、`2play-plan-101`、`agent-itinerary-100`。

## 2026-09-10 — UI 用语：框架 ≠ skeleton（骨架）

- **范围：** `06-plan-assistant-t3.html`、`mockup.css`（助手 bubble 对比度）、`2play-design` §4.7.1、`2play-plan-101`、knowledge `ui-framework-vs-skeleton.md`。
- **变更：** 用户可见「骨架」一律改为「框架」；就绪句改为 `{目的地}{天数}天{人数}人{类型}行程框架已经规划完毕：`；助手说明改用 `.bubble--agent`；禁止 UI 再写「骨架」。
- **关联：** ADR-062、`2play-plan-101`。

## 2026-09-10 — MVP-T3 助手 mock（无四问 · 进度 · 骨架）

- **范围：** `ui-mockup/06-plan-assistant-t3.html`、`mockup.css`（`.plan-progress*`）、`06-plan-qa.html` 归档跳转、gallery / takeoff-11 提交链、`2play-design` §4.7.1、`2play-plan-101` Mock 指针。
- **变更：** 起飞提交后助手接管；友好阶段步骤（非固定四问）；主区 as-built 骨架；终止/composer/快捷操作集中在助手；`06-plan-qa` 固定四问废止。
- **关联：** ADR-062、`2play-plan-101`。

## 2026-09-10 — MVP-T3 AC + 测试矩阵（ATDD）

- **范围：** `2play-plan-101`、`agent-itinerary-100`、`2play-test-plan` §13、`agent-test-plan` §40、`plan.md`、`product-backlog`、`2play-design` §4.7.1、`agent-design` MVP-T3、ADR-062 指针。
- **变更：** 完整 GWT/AC（cleanup / 助手接管无四问 / trip_id / 进度 i18n / 骨架 UI / debug plan+pool；agent 骨架先行与阶段信号）；测试矩阵 Red；状态 **AC Ready**（待实现）。
- **关联：** ADR-062、ADR-050、ADR-056。

## 2026-09-10 — ADR-062：MVP-T3 skeleton vs T4 nominate/chat

- **范围：** `plan.md`、`product-backlog.md` §0/§1、`2.architecture.md`、ADR-061/062。
- **变更：** 确认下一 MVP = T3（cleanup + 助手接管 + `plan_trip`/`trip_id` + 进度文案 + 骨架 as-built UI + 复用 `/debug/plan`）；**废止**提交后固定四问；必去提名+聊天 refine → T4；原 T4–T7 顺延为 T5–T8。
- **关联：** ADR-062、`2play-plan-101` / `agent-itinerary-100`（后同日升为 AC Ready）。

## 2026-09-09 — MVP-T2 usable Confirmed

- **范围：** `2play-plan-100` DoD；plan/backlog/stories 状态。
- **变更：** usable Confirmed；下一开 MVP-T3。
- **关联：** ADR-061、knowledge `where2play-takeoff-constraints-mock-sync.md`。

## 2026-09-09 — Takeoff/Constraints mock → specs → code 对齐

- **范围：** `2play-design` §4.2.1/§4.7、`2play-stories`（`2play-plan-100` AC1、plan-46 AC12）、`2play-test-plan` §12、`where2play` takeoff/constraints/i18n/CSS。
- **变更：** 以 `06-plan-takeoff-11.html` + `06-plan-qa.html` 为 SoT：两行七列起飞轨；约束 11 项共享四列 + intake span-2；字段名（行程开始日期/每日起点/每日出发时间等）；无必去行；无 ± stepper。
- **关联：** ADR-061、`2play-plan-100`。

## 2026-09-09 — MVP-T2 specs + mock（Takeoff 11 → submit）

- **范围：** `2play-plan-100`、`agent-geocode-100`、2play design/test-plan、agent-design T2 节、`06-plan-takeoff-11.html`、ADR-061 Accepted（T2 条款）。
- **变更：** 11 字段 GWT/AC；geocode `{country,city,city_en?}`；mock 画廊链接；测试矩阵 §12。实现见同日代码落地。
- **关联：** ADR-061、`2play-plan-100`。

## 2026-09-09 — MVP 重切：Takeoff 11→submit（T2）与 after-submit（T3）

- **范围：** [`plan.md`](./plan.md)、[`product-backlog.md`](./product-backlog.md) §0/§1、[`tmp-0909.md`](./agent-specs/tmp-0909.md)、[ADR-061](./adr/ADR-061-takeoff-11-fields-skeleton-first.md)、architecture ADR 表。
- **变更：** 新产品切片 **MVP-T2** = 起飞栏 11 输入到提交（`2play-plan-100`）；**MVP-T3** = 提交后助手 + skeleton-first `plan_trip`（`2play-plan-101` / `agent-itinerary-100`）；原 T2–T5 顺延为 T4–T7。
- **关联：** ADR-061、ADR-060、ADR-050。

## 2026-09-09 — MVP-T1 收尾：设计合并进 agent-design

- **范围：** `mvp-1t-closing-plan.md`、`agent-design.md`、链接纠偏、冗余稿删/stub。
- **变更：** Item 1 usable Confirmed；`tmp-0909` + `real-agent-refactory` 真源并入 `agent-design`（真智能体能力/原则 + MVP-T1 as-built：逐题 session、芯片 collected 可空、起飞不强制 L3）；去重 T1 节；ADR-050 / architecture / backlog / stories 指针改锚 `agent-design`；`tmp-0909` 删除；`real-agent-refactory` / `draft-nominate-must-see-prompt` 改为短指针。
- **关联：** ADR-050、ADR-060、`agent-itinerary-93a` / `96`。

## 2026-09-09 — 出行限制 4 列网格

- **范围：** `plan-constraints` mockup + where2play；`2play-plan-90a` AC12。
- **变更：** 只读约束条改为 4 列上标下值：起飞 8 项两行（类型在预算前），横线下 intake 一行四格；必去点 chip。产品与 `06-plan-qa` 对齐。
- **关联：** `plan-constraints-panel.tsx`、`plan-intake.ts`、`ui-mockup/06-plan-qa.html`。

## 2026-09-09 — tmp-0909：起飞 2play↔agent 时序与数据流

- **范围：** `specs/agent-specs/tmp-0909.md`。
- **变更：** 按 ADR-060 重写：起飞 `plan_trip` intake 方法链、时序图、数据流图、后 4 问交互、与 L3/全环边界；去掉过时「强制提名 / 提示词 A/B/C」叙述。
- **关联：** ADR-060、`agent-itinerary-96`。

## 2026-09-09 — intake 去掉强制提名（纯环出芯片）

- **范围：** `plan_trip` intake、`agent-itinerary-96`、2play Q3、ADR-060。
- **变更：** 删除 `runForcedNominate`；芯片仅环内 `search_places`/`commit_trip`；同 trip 复用已有 must_see；Q3 空态 i18n + 手输 + 再 fetch。95 AC1 superseded。
- **关联：** [ADR-060](./adr/ADR-060-intake-no-forced-nominate.md)、`tmp-0909.md`。

## 2026-09-09 — L3 提名提示词收紧 + 落地跨语种 token 匹配

- **范围：** `buildNominateMustSeeUserMessage`（CN/HK/TW/EN）、`pickNominatedGroundCard`、`groundNominatedNameOnce` suggest 候选过滤。
- **变更：** 近郊改「目的地近郊有可安排一日游/半日游的知名必去地时，应至少列入一个」（时间预算+质量门槛+保底一个）；落地加 `sharedProperToken` 回退——子串不命中时按共享专名 token（≥4 字符、去通用场馆词）匹配，修 `Mosteiro dos Jerónimos`↔`Jerónimos Monastery` 跨语种别名。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`、ADR-042。

## 2026-09-09 — L3 提名提示词收紧（近郊/地理多样/非点）

- **范围：** `buildNominateMustSeeUserMessage`（CN/HK/TW/EN）。
- **变更：** 近郊由「也可」改「应至少列一个」；多样由「不要同一类」改「不要全集中在同一片区域」；加「不要线路/活动/打包描述」。措辞收紧，不加长。探针杭州 FAIL→PASS（近郊芯片出现）。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去门诚实化 + 一日游宽搜 + 拒绝规则合并

- **范围：** 探针近郊门、`groundNominatedName`、`pickNominatedGroundCard`、`nominateMustSeeViaLlm`。
- **变更：** 近郊门改为距锚点 >20km（不再用簇键启发式；20km 为近郊/一日游门槛，Sintra 23km 算）；落地加去后缀宽搜回退（>15km）；拒绝规则收成 `isNoiseCategory` + `isVagueAreaName`；suggest 有 tip 才重试；同 `native_id` 去重；探针 `attemptable` 分母对齐 `isVagueAreaName`。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去落地：suggest 优先 + 短名芯片

- **范围：** `groundNominatedName`、`mustSeeChipLabel`、`capClusterOccupancy`、双城探针。
- **变更：** 提名先 `suggest_places` 再同名 `search_places`；芯片 label 用提名短名；同簇最多 3；探针门加垃圾店/同簇/近郊。
- **关联：** `tmp-0909.md`、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — 必去提名 L3：JSON 短名 + 季节词

- **范围：** `buildNominateMustSeeUserMessage`、`parseNominatePlaceNames`、`pickNominatedGroundCard`。
- **变更：** 只输出 JSON 短地名；有 `bounds` 时拼月份+季节；落地跳过住宿/购物/整湖泛名。探针杭州 AMAP + 里斯本 GMAP 过门。
- **关联：** `tmp-0909.md`、ADR-059、`hangzhou-4d-amap-trueagent-probe.md`。

## 2026-09-09 — ADR-059 规划必须传入已知条件

- **范围：** 必去提名 L3；`plan_trip` / `discover_places` 转发 `bounds` 等。
- **变更：** 已有日期、起点名、用户必去、其他要求写入提名参数行；禁止因「只要天数」丢掉日历。不写城×季节表。
- **关联：** [ADR-059](./adr/ADR-059-pass-all-known-trip-constraints.md)、`places-ontology.ts`、`agent-itinerary-95` AC1。

## 2026-09-09 — 起点补全产品验收通过

- **范围：** `2play-plan-96` AC5。
- **变更：** 杭州 + `SFEE` 产品路径用户确认通过。
- **关联：** `knowledge/maps/origin-autocomplete-amap-gmap-probe.md`。

## 2026-09-09 — 起点：suggest_places 优先再 search

- **范围：** 起点验真（`2play-plan-96` AC5 / Story 5 AC16）、agent `suggest_places`、ADR-053 查找路径。
- **变更：** 一律先厂商 autocomplete（AMAP inputtips / Google places:autocomplete），按目的地收窄后再 hydrate/`search_places`；提示空才搜点。不分字种/字母长度。探针：`amap-sfee-origin-probe`、`origin-autocomplete-amap-gmap-probe`。
- **关联：** `plan-resolve-origin.ts`、`places-agent` adapters + `/v1/suggest_places`、ADR-053 D1/D4。

## 2026-09-09 — 起点验真线程：部分/零匹配文案

- **范围：** `2play-plan-96`、助手 hotel 题。
- **变更：** 精确匹配进下一题；部分匹配只显示 candidates 文案 + 纵向芯片；零匹配只显示 not_found；二者都不显示 `need_prompt.hotel`；再搜先清上一轮候选。
- **关联：** `2play-design.md` §4.12 Story 5。

## 2026-09-09 — 助手过程芯片进线程 + 约束条 mockup

- **范围：** 助手 1–3；规划栏仅 spec mockup。
- **变更：** 住宿/必去芯片进对话线程；dock 只留跳过/重答；去掉底部跳过提示；左上拉手外拉变大。`06-plan-qa` 约束条改为 4 列 + 宽格。
- **关联：** `plan-assistant-nav.tsx`、`plan-nav-resize.ts`、`2play-plan-97` AC5。

## 2026-09-09 — MVP-T1 故事 AC 签收

- **范围：** `93a` / `90a` / `96` / `97` / `98` / `99`。
- **变更：** 对照测试与代码签收为 Done；补 `2play-plan-99` GWT；`90a` AC2 写清 T1 不二次 `plan_trip`。产品 usable 仍待人确认。
- **关联：** `agent-stories.md`、`2play-stories.md`、`product-backlog.md`、`plan.md`。

## 2026-09-08 — Debug 页列出地图 source

- **范围：** `2play-plan-98` debug 表。
- **变更：** 候选/registry 从 card `provider` 或 `sources[]` 取供应商；`list_destination_pois` 回传 provider；debug 用 `play.plan.provider.*`；增加 Origin/stay 行。
- **关联：** `plan-discover-pool.ts`、`dispatch.ts` list_destination_pois。

## 2026-09-08 — 约束条对齐 8 字段起飞 + intake 回写

- **范围：** `plan-constraints` / `2play-plan-90a` AC12。
- **变更：** 字段序改回 mockup（类型/节奏/交通在住宿前）；节奏/交通用目录 i18n；agent 四问写入约束条。
- **关联：** `ui-mockup/06-plan-qa.html`。

## 2026-09-08 — 必去芯片：服务闸 + 8 条 + 排版

- **范围：** `agent-itinerary-95` AC2/AC6、`2play-plan-97` AC5。
- **变更：** L0 丢停靠点/游船/码头等服务类卫星 POI；L2「可停靠」改为「可落到地图上的一个点」；`MUST_SEE_LIMIT` 5→8；必去芯片纵向一行一点；跳过/重答钉在输入框上方左侧。
- **关联：** ADR-042（目的地无关模板，不扩 CATALOG）。

## 2026-09-08 — L0–L3 必去合同落地

- **范围：** `agent-itinerary-95` / 95b / 95c、`2play-plan-99`；稿 `draft-nominate-must-see-prompt.md` Accepted。
- **变更：** 设计并入 L0 无坐标丢弃、L1 工具一句、L2 系统本体、L3 四语可落点短句（近郊不限一处）；字段表不进提名 user；稳定 key 传 agent；同一 trip 换 locale 不重提名。
- **关联：** ADR-042、ADR-050。

## 2026-09-08 — 必去提名纳入行程类型 + 10 城审核

- **范围：** `buildNominateMustSeeUserMessage`；审核稿 `nominate-must-see-10city-audit.md`。
- **变更：** 提名提示使用已收集 `trip_type`/pace/budget；合适时含一日游；无城市 POI 表。10 城 × 中英直连名单供人审。
- **关联：** ADR-042、`agent-itinerary-95`。

## 2026-09-08 — T1 质量：提名去重、起点满匹配、助手导航、stop 源

- **范围：** `agent-itinerary-95`、`2play-plan-96`、`2play-plan-97`、`2play-plan-98`（计划稿曾用 93b/90b/90c/91，与 T2/T3 id 冲突故改号）。
- **变更：** backlog/stories/design 写入 GWT；intake 强制 LLM 提名+落地+族去重；起点仅满匹配唯一才 auto-hit；重答/跳过快答；debug 与行程标 Google/高德。
- **关联：** ADR-042、ADR-050、ADR-053。

## 2026-09-07 — T1：plan_trip 写 session、debug 池、registry 芯片、酒店跳过/验真、不自动 fill

- **范围：** `2play-plan-90a` T1 切片（不含 T2 骨架密度）。
- **变更：** `POST /api/plan/trip` 成功后 upsert `planSessionCache`；debug 页在 focus/轮询下重拉 current 与 stops-pool；fixture/空搜时 `must_see` 芯片取自城市 AttractionPoi；酒店空=skip、非空 422 不吞；四问后不再 `runPlan` NDJSON。
- **关联：** `2play-plan-90a`、`where2play/app/api/plan/trip/route.ts`、`places-agent/src/core/plan-trip.ts`。

## 2026-09-07 — 真智能体合规：芯片权威 + 省略 providers + 芯片走 plan_trip

- **范围：** must_see 由 LLM 提名落地卡标记；2play 产品路径省略 `providers[]`；discover 改 `plan_trip` + `fetch_trip_details`。
- **变更：**
  - `applyNominatedMustSee`：芯片 = 落地提名；heat 只补缺与同名平手。
  - `startPlanDiscover` 调 `plan_trip`，不再 `discover_places`。
  - `/api/plan`、discover、must-see-suggestions、travel_tips 不再拼 `providers[]`。
  - 前缀族去重仅作搜噪闸；去掉会误并「纪念馆」的后缀。
  - `pickLodgingStayCard` 与 2play 对齐：山庄等关键词 + 唯一 token 命中。
- **关联：** ADR-042、ADR-050、ADR-052、real-agent-refactory G6/G7。

## 2026-09-07 — 地图路由质量修复（大陆禁 Google 回退 + drive_preferred + 交通 UI 结构化 + LLM 必去提名 + 前缀族去重）

- **范围：** places-agent 与 where2play 地图路由质量问题修复，覆盖起点验真、必去点质量、排餐供应商、交通引擎、交通 UI。
- **变更：**
  - **起点验真**：`looksLikeLodging` 扩住宿关键词（山庄/公寓/民宿/别墅/hostel/bnb 等）；`ORIGIN_CANDIDATE_LIMIT` 3→6；新增 token-priority 匹配（用户 query token 唯一覆盖候选时直接命中，绕过 category 判定）。
  - **大陆禁 Google 回退**：`shouldTryGoogleAfterEmptyAmap` 无条件返回 `false`（景点/餐厅/交通）。
  - **排餐**：AMAP 餐厅搜索半径 `DINING_AROUND_RADIUS_M` 1000m→3000m。
  - **交通引擎**：`ItineraryPreferences` 新增 `drive_preferred`；`transitPreferred` 正则补 "交通"；AMAP Directions 调用传 `city` 参数（修复非上海城市默认上海）；坐标匹配优先 `native_id` → 精确名 → 归一化名；失败标记 `transit_outcome: "partial"`。
  - **必去点**：`nominateMustSeeViaLlm` LLM 提名 3-5 必去点（含 day-trip）→ `search_places` 落地坐标。
  - **前缀族去重**：`attractionClusterKey` 剥离 "景区"/"售票处"/"重建记" 等卫星后缀，避免同一地标（如雷峰塔）重复推荐。
  - **交通 UI**：`ItineraryTransitSlot` 扩展 `from` / `to` / `legs[]` / `outcome`；行程详情页结构化渲染交通药丸（transit-from / transit-to / transit-option）。
- **关联：** [ADR-052](./adr/ADR-052-map-provider-routing.md)（D4/D7/Consequences 修订）；`plan-resolve-origin.ts`、`tools.ts`、`amap/direct.ts`、`itinerary-timed.ts`、`plan-agent-body.ts`、`plan-enrich-transit.ts`、`itinerary-planner.ts`、`discover-dedupe.ts`、`itinerary-types.ts`、`itinerary-skeleton-map.ts`、`plan-skeleton-fill.ts`、`plan-itinerary-view.tsx`。

## 2026-09-07 — MVP-T1 开工（need_input 4 题 + 大陆 seed + 开关机制）

- **范围：** `agent-itinerary-93a` 问题集；测试策略写清 seed/真 API 开关；大陆 4 城 stops pool。
- **变更：**
  - `defaultNeedInput`：`hotel` / `start_time` / `must_see` / `other`；`ask_user` 保留 options；collected 回填 must_see 芯片。
  - [`agent-test-plan.md`](./agent-specs/agent-test-plan.md) 增 §1.3 开关机制 + MVP-T1 用例。
  - [`seed-city-pois.ts`](../places-agent/scripts/seed-city-pois.ts) 加杭州/西安/上海/厦门；大陆 AMAP 文本分页。
  - Probe 文件名加 sha1，避免 CJK geocode 串城（见 knowledge `probe-cache-cjk-filename.md`）。
  - UI mock：[`06-plan.html`](./2play-specs/ui-mockup/06-plan.html) 8 字段；[`06-plan-qa.html`](./2play-specs/ui-mockup/06-plan-qa.html) 4 问逐题。
- **关联：** ADR-052、ADR-057、ADR-058；`plan-trip.ts`。

## 2026-09-07 — POC 签收（agent-poc-01 Done）

- **范围：** 真智能体 POC 标 Done；工作计划切到 MVP-T1。
- **变更：**
  - [`product-backlog.md`](./product-backlog.md) §0 / §1：`agent-poc-01` → Done；下一步 MVP-T1。
  - [`plan.md`](./plan.md)：POC 待办全部勾选；下一步改为写 MVP-T1 AC。
  - [`agent-stories.md`](./agent-specs/agent-stories.md)：`agent-poc-01` 状态 Done；范围注明全环超出原 ADR-054 D2。
  - [`real-agent-refactory.md`](./agent-specs/real-agent-refactory.md)：POC 签收行改为已完成。
  - 验证脚本补质量 mock（时钟链 / 下午站 / 餐店卡）与检查项；产物 [`poc-true-agent-verification.html`](./poc-true-agent-verification.html)。
  - 知识：[`knowledge/agent/poc-verify-fixture-quality.md`](./knowledge/agent/poc-verify-fixture-quality.md)。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`places-agent/scripts/verify-poc-true-agent.ts`。

## 2026-09-07 — POC 全环模型驱动 + 无 Google 验证

- **范围：** `plan_trip` 全环改为模型 act-or-stop；fixture 验证不调 Google。
- **变更：**
  - `runFullLoopAgent` 为默认全环：模型选 `resolve_origin_stay` / `search_places` / `make_itinerary` / `plan_next_stop` / `commit_artifacts` / `stop`；全环发现的 eligible POI 回填 registry。
  - 旧 `runFullLoop` 固定管线保留为 `PLAN_TRIP_LEGACY_FULL_LOOP=1` 回退。
  - 新增 `places-agent/scripts/verify-poc-true-agent.ts`（进程内、fixture、Prisma Lisbon 池）；结果写入 [poc-true-agent-verification.md](./poc-true-agent-verification.md)。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) G8 行标记已实现。
- **关联：** ADR-054、ADR-056、ADR-057、ADR-058；`plan-trip.ts`。

## 2026-09-07 — 跨供应商同地点去重交由 LLM（ADR-058）+ 真智能体改进路线

- **范围：** stops pool 跨供应商去重策略；POC 真智能体合规路线。
- **变更：**
  - 新增 [ADR-058](./adr/ADR-058-cross-provider-duplicate-llm-judges.md)：跨供应商同一地点（香港 Google+AMAP）不写库合并、不读池硬去重，去重由环内 LLM 取点时自行判断；prompt 硬约束 + 事实闸兜底。
  - 更新 [agent-design.md](./agent-specs/agent-design.md) §9.2 / §5.2：明确 LLM 取点需自己判断跨供应商同地点。
  - [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 追加「下一步改进」节：列出 G6/G7/G8/F89/G5 + POC 签收待决项、建议顺序、结合 ADR-057 三层测试的验证矩阵。
- **关联：** ADR-049、ADR-050、ADR-052、ADR-056、ADR-057、ADR-058；`destination-poi-registry.ts`；`plan-trip.ts`。

## 2026-09-07 — 成本节约测试策略入库 + change-log 启动

- **范围：** agent 测试策略文档化；修订记录机制启动。
- **变更：**
  - 新增 [ADR-057](./adr/ADR-057-cost-conscious-agent-test-strategy.md)：L0 fixture CI、L1 probe 文件缓存、L2 日配额门、stops pool 作 live feed（非城市百科）。
  - 更新 [agent-test-plan.md](./agent-specs/agent-test-plan.md)：绑定表、§1.2、§9 CI 门控、§11 清单。
  - 新建本文件，作为此后 specs 修订的单一入口；[`README.md`](./README.md) Layout 表增加 `change-log.md` 行。
- **关联：** ADR-021、ADR-042、ADR-056、ADR-057；`.github/workflows/places-agent-tests.yml`；`places-agent/scripts/seed-city-pois.ts`、`probe-budget.ts`、`search-cache.ts`。

## 2026-09-07 — Registry 回填语义 + 三城 stops pool 种子

- **范围：** AttractionPoi 写路径与测试 feed。
- **变更：**
  - 新增 [ADR-056](./adr/ADR-056-registry-backfill-semantics.md)：唯一性 `(provider, native_id)`；匹配无差跳过 / 有差更新+aliases / 无匹配新增；`cardSlim` 存 photos、不存 `must_see`；`plan_trip` commit 接线 `safeUpsertEligiblePois`。
  - 更新 [real-agent-refactory.md](./agent-specs/real-agent-refactory.md) 挂钩表与落库节。
  - 种子脚本建 Lisbon / Hong Kong / Taipei 池各 ≥100（有图优先）；删除旧「里斯本」「香港」碎片 destination。
- **关联：** ADR-049、ADR-051、ADR-056；`destination-poi-registry.ts`、`plan-trip.ts`。
