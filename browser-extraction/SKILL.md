---
name: browser-extraction
description: Rules for writing efficient browser-based UI extractors and parsers to minimize memory usage and token consumption.
---
# Browser Extraction Guidelines
1. **DOM Scoping**: Avoid parsing the full DOM. Restrict parsing dynamically to target structural elements like `nav`, `header`, `main`, and `footer`.
2. **Element Capping**: Extraction logic should aggressively limit the number of semantic elements returned (e.g., max 80-100).
3. **Playwright Management**: Use `@crawlee/browser-pool` and ensure `retireBrowserAfterPageCount` is set (e.g., max 50) to avoid memory accumulation in persistent browser pools.
4. **Resilience**: Implement intelligent fallbacks (e.g., Browserbase) for bot-protected sites or timeouts.
