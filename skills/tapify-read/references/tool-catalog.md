# Tapify tool catalog

Current catalog: **31 read-only tools**, conditional on reader availability and
explicit effective scopes. This is a map of existing capabilities, not authority
to upgrade any connection. Every HTTP request re-evaluates authorization; API
permissions, membership, selected resources and masking remain authoritative.

| Tools                                                                          | Required effective scope         | Coverage and interpretation                                                                                                                                                                                                |
| ------------------------------------------------------------------------------ | -------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `get_my_access`                                                                | `tapify.stations.read`           | Current business, approved station IDs and effective read capabilities.                                                                                                                                                    |
| `list_stations`, `list_locations`, `get_station`                               | `tapify.stations.read`           | Selected stations only; locations is a station alias, not the public/menu directory. No station keys.                                                                                                                      |
| `list_taps`, `get_tap`                                                         | `tapify.taps.read`               | Selected-station taps; active/maintenance flags are configured state, not heartbeat/live telemetry.                                                                                                                        |
| `list_sessions`, `get_session`                                                 | `tapify.sessions.read`           | Selected-station dispensing, mL, no customer/card/wallet data. List start-time window is at most 31 elapsed days.                                                                                                          |
| `list_kegs`, `get_keg`, `list_products`                                        | `tapify.inventory.read`          | Current approved-station attachments only; detached/reserve stock excluded. Products require one station and omit prices/media/translations/counts.                                                                        |
| `list_keg_inventory`, `get_inventory_keg`, `list_keg_events`                   | `tapify.inventory.business.read` | Whole selected business non-deleted keg stock, including detached stock and wider attachments; history parent is keg ID. Recorded volume/history quantities in mL.                                                         |
| `list_product_catalog`, `get_product`                                          | `tapify.inventory.business.read` | Whole selected business non-deleted catalog, descriptions/translations/media/template reference. Catalog price is RON bani **per litre**, 100 bani per RON. Alcohol is % ABV.                                              |
| `list_cards`, `get_card`, `list_card_transactions`                             | `tapify.cards.read`              | Business-associated cards and only this business's balance/ledger. RON bani, 100 per RON. History parent is card ID. Masked UID/customer fields remain masked; no NFC payload/signature.                                   |
| `list_customers`, `get_customer`                                               | `tapify.users.read`              | Customers related through business dispensing/card credit, not a global directory. Existing CLIENT/OPERATOR customer masking.                                                                                              |
| `list_operators`, `get_operator`                                               | `tapify.operators.read`          | Staff assigned to this business, no credentials or permission mutations. Operator contact projection differs from masked customer views; honor returned masking flags.                                                     |
| `get_sales_summary`                                                            | `tapify.analytics.read`          | Approved stations, settled wallet usage charges, mL, at most 31 elapsed days. Not fiscal revenue, refunds/top-ups or profit.                                                                                               |
| `get_sales_by_day`, `get_sales_by_hour`                                        | `tapify.analytics.read`          | Same settlement semantics, at most 31 elapsed days; explicit UTC/Europe-Bucharest calendar buckets, including empty buckets and distinct repeated DST hours.                                                               |
| `get_sales_trend`                                                              | `tapify.analytics.read`          | At most 366 elapsed days, clipped UTC calendar-month buckets; optional immediately preceding equal-duration comparison, not calendar-year alignment. Zero-baseline percent change is null.                                 |
| `get_digital_menu`, `list_menu_categories`, `list_menu_items`, `get_menu_item` | `tapify.menu.read`               | Active business-wide menu, independent of stations; no archived/inactive categories/items. Menu price is hundredths of configured currency **per item**, not dispensing RON/L. Original language, configured availability. |

## Inputs, ordering and completeness

- All list pages: `take` 1–100, default 20; exclusive ascending `afterId` from
  `nextCursor`. Keep filters unchanged; continue until null. ID order is not
  menu `sortOrder` or history event-time order. Pages are not an atomic snapshot.
- Detail IDs: `stationId`, `tapId`, `sessionId`, `kegId`, `itemId` as appropriate;
  business detail tools use `id`. Use IDs from the corresponding authorized list.
  Actor/user/business selectors are not accepted as authorization inputs.
- Tap/session/attached-keg/sales queries optionally accept an approved `stationId`;
  omission means approved stations, not business-wide coverage. Attached product
  lists require it. Business-wide inventory deliberately has no station filter.
- Business list `search`: optional trimmed 2–80 character substring. Kegs:
  UID/product name; catalog: product name; cards: UID; customers/operators:
  name/email/nickname. Search filters stored values but never removes masking.
- Business history: required `parentId`, cursor/take, no date/search filter.
- Menu lists require `menuId` from `get_digital_menu`; item lists optionally
  filter `categoryId`. Null menu data means no active menu.
- Session windows: UTC `Z` `[createdFrom, createdBefore)` by start time.
  Sales windows: UTC `Z` `[settledFrom, settledBefore)` by settlement time.
  Resolve local dates first, converting each boundary separately. Day/hour
  timezone is `UTC` (default) or `Europe/Bucharest`; choose explicitly. Local DST
  days may last 23/25 hours; repeated labels require UTC ranges/offsets.

## Output and cost limits

Volumes use mL (divide by 1000 for litres). Keg-event null quantities mean not
recorded. All money cents fields use 100 minor units per major unit; catalog
dispensing price is per litre, menu price per item. Ledger signed amounts require
their type. Observed/updated times are API read/record times, not device telemetry.

Transport accepts at most **16 MiB (16,777,216 bytes) per API-read JSON body**,
with a **3-second upstream timeout**. Identity receipt/delegation and denial-body
parsing retain a **128,000-byte** ceiling. Size is checked using declared length
and streamed UTF-8 bytes. A maximum 65,535-character description with 30 equally
large translations fits as one upstream product even with six-byte JSON escaping;
it does **not** fit the stricter server output budget. Large pages may also fail.
MCP preserves both JSON text and structured content on success. The server checks
their combined serialized tool result against **1 MiB (1,048,576 bytes)** and
bounds the entire MCP HTTP response, including framing, to **2 MiB (2,097,152
bytes)** before sending it. Overflow returns a small actionable error instead of
a truncated/partial success. These are server capacity limits, not host promises.

The server admits at most four MCP requests globally to work, with no queue, only
after the body is parsed and the token verifies; uploads and verification must
finish within four seconds. A ten-second arrival-to-completion lifetime covers
the whole request, including slow response writers. Disconnects/deadlines cancel upstream work and release the
permit; saturation is `503 gateway_busy` with `Retry-After: 1`. Send one operation
per request, not a batch. Host context/output limits remain independent and
require actual-host tests.

There is no truncation or automatic retry. Oversized reads return
`response_too_large`, not an empty/successful partial page. Retry the same cursor
with smaller `take`, down to 1; if a single record still fails, contact support
with the request reference. Do not skip a failed page or claim completeness.
For expiry retry once as a new request, then reconnect if persistent. For missing
access check approved context/current product permissions; reconnect with explicit
intended consent. For quotas wait; for transient outages retry once after a short
wait. Persistent failures need support; raw payloads/tokens stay out of reports.
