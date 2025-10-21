# Communication & Language Guidelines (VN/EN)

Purpose: Keep collaboration smooth when mixing Vietnamese and English, while helping the team improve English over time.

## Principles

- You can mix Vietnamese and English in discussions and tickets.
- Default assistant replies in English, but will interpret Vietnamese terms directly.
- On first mention of a Vietnamese term, provide a concise English equivalent in parentheses, e.g., "phí chiết khấu (commission fee)".
- Keep product/brand names as-is (e.g., Zalo Mini App, MoMo, VNPay).
- If a term is ambiguous, ask one brief clarifying question and propose 1–2 English options.
- Optionally include a short "Glossary" section at the end of messages when new terms appear.

## Documentation Rules

- Source docs in English by default; include Vietnamese examples/snippets where clarity improves.
- For UI copy, maintain vi-VN as primary with en as secondary.
- In data models, provide bilingual fields when needed (vi/en) for titles, descriptions, and labels.

## Style Notes

- English tone: clear, friendly, and professional. Avoid slang that may not generalize.
- Vietnamese tone: thân thiện, chuyên nghiệp, dễ hiểu toàn quốc (tránh từ lóng vùng miền).
- Use Vietnamese locale defaults (vi-VN) for formatting dates, numbers, currency (₫).

## Implementation in Workflows

- When writing PR descriptions or issues, feel free to include Vietnamese terms; add English clarification in parentheses at first use.
- For code identifiers, prefer English. For user-visible text, localize to vi-VN first.
- Maintain and update the shared glossary for recurring terms.
