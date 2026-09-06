# ADR-051: discover / fill 解析可展示图地址并写入候选与停点

## Status

**Accepted**（2026-09-06）— 产品确认：缩略图真源在 places-agent 建池/填站时解析并落库；where2play 不另做取图。**实现：** `resolve-display-photo.ts` + discover / `plan_next_stop`（餐）挂点（2026-09-06）。

**补丁**（2026-09-06）— 起点 stay 图曾挂在 `plan_next_stop`。**再补丁（同日）：** 有 [ADR-053](./ADR-053-origin-stay-as-stop-card.md) 指针时 **intake 建卡时解析**，fill 只抄；无指针才允许 fill 现搜（禁 `cards[0]`）。2play / fetch 仍不取图。

**补丁**（2026-09-06）— 高德搜索常给 `http://store.is.autonavi.com/...`。D3/D4 的 https 门与浏览器混合内容会剥掉大半大陆景点图。**只对高德 CDN 主机升 https**（D6）。

## Context

where2play 行程站缩略图（尤其 Lisbon / Google）为空。叠加原因：

1. Google Places 搜索只给 `photos[].name`，不是浏览器能开的图。
2. Agent 曾把带 `key` 的 media URL 写入 `photos[]`，随后 `sanitizePublicUrl` / slim **剥掉 key**，账本里剩打不开的 `.../media`。
3. 2play 用该 http 当 `<img src>`，并曾用 `get_place_details` 回填——仍是同一类不可展示地址。
4. 曾讨论 BFF 照片代理、或让 `fetch_trip_details` 取图落库。后者违反 [ADR-046](./ADR-046-trip-store-pg-memory-fetch.md)（fetch **只读**）。
5. **起点 stay** 不进 `discover_places` 景点池（[ADR-049](./ADR-049-verified-attraction-and-meal-slots.md)）；`candidates.stays` 多为酒店名。填站时 `stop_display` 若无 PlaceCard，则无 `photos[0]`，列表 `stop-origin` 拇指为空。Intake 的 `search_places` 只定店与坐标，**不是**图片真源（避免 intake / fill 双真源）。
6. **高德 `photos[].url` 常为 `http://`。** 主机多为 `store.is.autonavi.com`（`showpic` / `query_pic`）。`isDisplayablePhotoUrl` 只收 `https://`；`slimArrangeCandidate` 同样只抄可展示 URL。杭州实测：同一批搜索约一半图链是 http，解析后账本无图。任意 http 仍禁止（混合内容 / 非高德主机不可信）。

what2eat 已有补图链：Google Maps 无图 → 其它 Google 服务 → TripAdvisor。places-agent 已持 Google 与 TripAdvisor 钥匙。服务端用 key 解析 **无 key 的 CDN `photoUri`** 见 [`knowledge/maps/google-photos-media-url.md`](../knowledge/maps/google-photos-media-url.md)。

## Decision

### D1 — 解析与落库跟「谁建卡」走

1. **景点：** `discover_places` 建 `candidates.places` 时解析 **一张** 可展示 `https`，写入 `photos[0]`（可选 `photo_source`），经 slim + `dualWriteTrip` 落库。`plan_next_stop` 填景点时 **从池抄到** filled stop。
2. **正餐（午餐/晚餐）：** 按 [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) 不进骨架停点、不靠 discover 景点池。店由 **`plan_next_stop` 现搜**。同一套解析挂在填站写卡时，写入该 meal stop（及本 trip 餐厅卡若落库）。
3. **起点 stay（含 `origin_mode` / day_origin / return / midday）：** 身份卡在 **intake 选定**时建齐（[ADR-053](./ADR-053-origin-stay-as-stop-card.md)），不由 discover 建池。填站：**抄** `originStay` / 停点上的指针与已有 `photos[0]`；有 `native_id` 或可展示图则 **不重搜**。仅无指针（skip / 失败仅 name）且尚无图时，才可按 **酒店核心名**（括号不进 query）+ 城市/near 搜住宿类卡，再跑同一套 `resolveDisplayPhoto`；**禁止** `cards[0]`。同 trip 复用已解析卡。2play **不**解析起点图。
4. where2play 不另「找图」。
5. `fetch_trip_details` **只读** 已写入的 `photos`。禁止在 fetch 上解析图或改 revision。

### D2 — where2play 不加取图方法

2play 只把账本/切片上的 `photos[0]` 映射为 `photoUrl` 给 `<img>`（含 `stop-origin`）。禁止：产品侧 Photo 代理当真源、为补图主路径调 `get_place_details`、源码城市图百科（[ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)）。

### D3 — 解析链（服务端，钥匙不进账本）

对每张 **当时正在写入的卡**（discover 的景点，或 `plan_next_stop` 的餐店 / **stay**），每卡最多一张：

1. 高德 `photos[].url`（已是直链）→ 用。若为 `http://` 且主机为高德 CDN（`*.autonavi.com` / `*.amap.com`），**升为 `https://`** 后再写入（否则 ADR-051 的 https 门与浏览器混合内容会剥掉大半大陆景点图）。
2. 否则 Google Places 搜索卡片 `photos[].name` → agent 用已有 Google key：`GET .../media?maxWidthPx=800&skipHttpRedirect=true` → JSON **`photoUri`**（CDN，无 key）。**800** = 原缩略请求宽度 ×2，供列表拇指与 place-sheet lightbox **共用**同一张 `photos[0]`（不另开第二 URL / 第二真源）。
3. 否则其它 Google（如 Place Details 的 photos，同一把 key，同宽度）。
4. 否则 TripAdvisor（已有 `TRIPADVISOR_API_KEY`）配图。
5. 都没有 → 不写 `photos`，不编造。

`GOOGLE_PHOTOS_ENABLED=false` 时跳过 Google 两步，仍可走高德 / TA。并发加上限，避免拖垮 discover / fill。

### D6 — 高德 insecure 直链只升已知 CDN

1. 写入前对 `photos[]` 跑 `upgradeAmapInsecurePhotoUrl`：仅当 URL 为 `http://` 且 hostname 为 `*.autonavi.com` 或 `*.amap.com` 时改为 `https://`。
2. 其它 `http://` 仍视为不可展示（与 D4 一致）。
3. 挂点：`amapPoiToCard`（搜索 / details 入库）与 `resolveDisplayPhoto` / `firstDisplayable`（discover slim 二次门）。不在 2play 升协议。
4. 无图时仍不编造、不把 Google 当大陆 discover 补图真源（[ADR-052](./ADR-052-map-provider-routing.md) 禁止 discover 扩源）。
5. 已落库的无图 / http 行程 **不回填**（同 D5）。

### D4 — 禁止写入带 key 的 media URL

账本、写信封、2play 可见 JSON **不得**含 `key` / `token`。`sanitizePublicUrl` 继续剥 key。最终 `photos[]` 必须是已解析的公开 https（高德 / `photoUri` / TA），不是剥了 key 的 `places.googleapis.com/.../media`。

### D5 — 旧行程

已落库的坏 URL 或缺图 stay **不回填**。景点须 **重新 discover**；起点 / 正餐须 **重新 fill**（新开一程或同等重跑 `plan_next_stop`）才有可展示图。

## Consequences

- **正：** 图跟「谁建卡」走（discover 景点 / fill 餐与 stay），与 ADR-046 读模型一致；2play 无第二真源；钥匙留在 agent；stay 与 meal 挂点对称。
- **负：** discover 与 stay/meal fill 多 Photo / 偶发 Details / TA 请求（仅首图）；旧 trip 要重跑 discover 和/或 fill。高德 CDN 升 https 后，旧无图行程仍须重跑 discover。
- **中性：** MVP 不强制 2play/agent 图片字节代理；若 CDN 热链失效，再开代理故事，不改变 D1–D4。Lightbox 与列表共用账本 `photos[0]`（`maxWidthPx=800`）；不另存大图字段。Intake 预览大图仍为可选后续，不改变本 ADR 真源。

## References

- [ADR-046](./ADR-046-trip-store-pg-memory-fetch.md) — fetch 只读
- [ADR-049](./ADR-049-verified-attraction-and-meal-slots.md) — 餐与 stay 不靠 discover 景点池
- [ADR-001](./ADR-001-thin-app-agent-split.md) — 薄应用
- [ADR-007](./ADR-007-tripadvisor-match.md) — TA 匹配
- [ADR-042](./ADR-042-no-city-encyclopedia-in-source.md)
- [ADR-048](./ADR-048-skeleton-geo-anchor-is-destination.md) — origin 解析边界
- [ADR-053](./ADR-053-origin-stay-as-stop-card.md) — 起点整卡；fill 只抄
- [google-photos-media-url.md](../knowledge/maps/google-photos-media-url.md)

## Date

2026-09-06（补丁：stay / origin 由 `plan_next_stop` 解析图 — 同日；再补丁：ADR-053 + `maxWidthPx=800`；再补丁：D6 高德 CDN `http`→`https`）
