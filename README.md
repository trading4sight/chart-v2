# chart-v2
TRADING4SIGHT is a Trading Chart Platform (ONLINE MODE : 16 JUNE 2026 - IN PROGRESS)

---
# Changelog

## 2026-06-17

### OpenAlgo CORS & WebSocket Deployment Fixes
- Added the `ngrok-skip-browser-warning` header to all OpenAlgo REST API POST requests in [client.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/openalgo/client.ts) to bypass the ngrok warning screen and prevent CORS blocks when deployed to remote environments (e.g. GitHub Pages).
- Fixed the WebSocket URL formatter in [wsClient.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/openalgo/wsClient.ts) to use `ws://` instead of `wss://` for local loopback addresses (like `127.0.0.1` or `localhost`). This allows browsers on HTTPS origins to successfully connect to the local WebSocket server since loopback is exempt from mixed-content blocking.
- Fixed a bug where the application got stuck on "Loading chart..." when deployed to remote/static environments (like GitHub Pages) due to the symbols manifest being empty. The check in `mountChartIfReady()` inside [main.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/main.ts) was updated to allow chart mounting to proceed in online mode regardless of the presence of local CSV symbols.
- Fixed a bug in [onlineLoader.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/onlineLoader.ts) where scroll-back paging was broken in online mode. Corrected the loader checks to use the `forward` load type rather than `backward` (since KlineCharts represents scroll-back as `forward`), and implemented `onlineHasMoreMap` to track server-side historical data availability. This ensures `more.forward` correctly triggers subsequent historical fetches past the initial 90-day page until server-side data is fully exhausted.
- Fixed a bug where the Volume Cluster overlay plotted for only 1-2 days on lower-TF charts (e.g. 5m chart) even when "History Range" was set to "6 months". Added `customDurationDays` parameters to `getOnlineCachedBars()` and `fetchCachedBars()` in [onlineLoader.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/onlineLoader.ts) to request the full settings range duration. Updated [volumeCluster.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/overlays/volumeCluster.ts) to check the resolved source history range and trigger a new fetch when settings are updated to a wider range.
- Added a built-in HTTP proxy router in the production distribution server template inside [vite.config.mjs](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/vite.config.mjs) so that running the built app locally (via `node dist/server.mjs` or `npm run serve:dist`) automatically forwards `/api` requests to the local OpenAlgo backend on port 5000, eliminating CORS issues for production builds.

## 2026-06-16

### TradingView Market Status & Timings Popover
- Implemented an interactive TradingView-style market status indicator next to symbol details in the legend using precise SVG icons (provided by the user) for pre-market, open, and closed states.
- Created `marketHours.ts` utility to compute market statuses ('open', 'pre-market', 'closed') and holiday schedules (2025-2026) for Indian exchanges (`NSE`, `BSE`, `NFO`, `BFO`, `CDS`, `BCD`, `MCX`).
- Designed a custom DOM component `SeriesLegend` to overlay the chart stage showing the symbol details, OHLC metrics, and an interactive status dot.
- Built a timezone-aware, responsive timings popover containing a visual timeline with markers (`Pre-open`, `Open`, `Close`), a real-time cursor indicator (when open), and support for both ticking countdowns (when open/pre-market) and static notifications (when closed). Updated the popover header to display the corresponding SVG status icon matching the current market state.
- Configured `symbolTitle` and `symbolValues` status line settings to default to `true` inside `createDefaultChartSettings()`, enabling the SeriesLegend overlay automatically on initial startup.
- Wired `SeriesLegend` into `ChartManager` and `MultiChartWorkspace` to support multi-pane charts and live hover coordinate updates.
- Added comprehensive styles in `style.css` supporting the white theme, status dot color variations, popover card design, and timeline tracks. Updated CSS rules to size the SVG icons at 18px, remove borders, and apply status colors.
- Modified the popover logic to conditionally close during component re-render only if the status dot element is not found in the DOM (e.g. legend hidden or component destroyed), resolving flickering on real-time chart ticks.

### Volume Source Selection via Search in Online Mode
- Implemented search-callback support in [SymbolSearchModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/SymbolSearchModal.ts) via a custom `onSelect` callback payload to allow selecting any symbol from online search without changing the active chart symbol.
- Integrated search capability into [getSourceSymbolOptions](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/config/symbols.ts) by displaying the currently active custom symbol and offering a "Search online symbol..." trigger dropdown option when in online mode.
- Wired search trigger and selection callbacks into the volume source select fields of all drawing overlay settings modals: [SessionVolumeProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/SessionVolumeProfileModal.ts), [FixedRangeVolumeProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/FixedRangeVolumeProfileModal.ts), [TpoProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/TpoProfileModal.ts), and [VolumeClusterModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/VolumeClusterModal.ts).
- Integrated search triggers into [IndicatorSettingsModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/IndicatorSettingsModal.ts) for indicator volume source selection (e.g., Volume YSTC) to fully support selecting alternate symbols via search in online mode.
- Fixed z-index layering in [style.css](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/style.css) by increasing `.symbol-search-modal` z-index to `50` so that it renders in front of settings modals (z-index `45`/`46`) instead of behind them.
- Fixed exchange resolution in [symbols.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/config/symbols.ts) for online-only derivatives (NFO/BFO) and indices (NSE_INDEX/BSE_INDEX) by implementing a smart `detectOnlineExchange` parser, preventing the "Volume source data unavailable" error on non-equity selections.
- Added missing `applySettings()` calls to `onSelect` callback triggers across all four technical overlay settings modals ([FixedRangeVolumeProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/FixedRangeVolumeProfileModal.ts), [SessionVolumeProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/SessionVolumeProfileModal.ts), [TpoProfileModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/TpoProfileModal.ts), and [VolumeClusterModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/VolumeClusterModal.ts)), ensuring custom online source symbol updates save and apply to the active overlays immediately.

### Online Historical Paging & Date Navigation
- Fixed a bug in [onlineLoader.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/onlineLoader.ts)'s cache logic where backward paging requests short-circuited when any bar older than the requested timestamp was found. It now only short-circuits if we have at least 100 bars older than the requested timestamp in cache, allowing scroll-paging to load earlier history seamlessly.
- Fixed a bug in [GoToDateModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/GoToDateModal.ts) where selected dates outside the currently loaded bar range were clamped to the earliest/latest loaded bar boundaries. It now allows returning the raw selected timestamp if it falls outside the range, triggering a new chart data window fetch to load older history from the server.

### Default Online Mode & NIFTY50-INDEX Startup
- Configured the application to default to `'online'` mode on initial boot in [main.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/main.ts) when no mode is present in `localStorage`.
- Set the default symbol selection to `NIFTY` on the `NSE_INDEX` exchange with the `D` (Daily) timeframe in online mode in [symbols.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/config/symbols.ts).
- Modified the main entry point's CSV checking logic in [main.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/main.ts) to only require local CSV file imports when starting in offline mode, allowing online mode to initialize and run perfectly on empty setups.
- Mapped `NIFTY50` to the correct OpenAlgo live symbol `NIFTY` in `toLiveSymbol` in [onlineLoader.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/onlineLoader.ts) to resolve the Nifty 50 Index correctly and avoid the Symbol 'NIFTY50' not found error on startup.


### Online Mode Timezone Fix
- Removed the `IST_OFFSET_SECONDS` (+5:30 hours) addition from online history bars in [onlineLoader.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/onlineLoader.ts) and live tick timestamps in [chartInit.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/chartInit.ts).
- OpenAlgo returns proper UTC epoch timestamps; KlineCharts already applies the configured `Asia/Kolkata` timezone for display via `chart.setTimezone()`. The IST offset was double-shifting timestamps, causing online candles to appear 5.5 hours ahead of correct market times.
- Online mode now shows timestamps consistent with offline CSV mode.

### OpenAlgo WebSocket Real-time Integration
- Implemented a unified OpenAlgo WebSocket Client in [wsClient.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/openalgo/wsClient.ts) to connect, authenticate with API keys, reply to heartbeats (`ping`/`pong`), auto-reconnect with exponential backoff, and manage subscriptions dynamically.
- Integrated WebSocket subscriptions with the chart data loader's `subscribeBar` and `unsubscribeBar` callbacks in [chartInit.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/chartInit.ts) to route live quotes (`tick:update`) to the active chart.
- Aligned real-time tick timestamps to current timeframe bar boundaries using UTC epoch milliseconds, consistent with how KlineCharts displays dates via `setTimezone('Asia/Kolkata')`.
- Wired WebSocket state transitions into [ChartManager.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/ChartManager.ts) to establish connection on startup or mode changes.
- Integrated real-time market data into the Order Panel in [OrderPanel.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/OrderPanel.ts), dynamically updating bid/ask prices on action/submit buttons and showing a premium, 5-row Depth of Market (DOM) table with relative volume bar backgrounds in the `DOM` tab.
- Added premium CSS grid styling and background relative bar gradient fills in [style.css](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/style.css) for the DOM table.

### Countdown to Bar Close
- Implemented the "Countdown to bar close" setting in [SettingsModal.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/SettingsModal.ts) as a dynamic checkbox bound to scales configuration.
- Added a `formatExtendText` callback in `buildChartFormatter` inside [chartSettings.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/chartSettings.ts) to compute the remaining duration for the current candlestick.
- Verified that countdown calculates correctly using UTC epoch timestamps in online mode, updates dynamically every second, and respects visibility toggles.

### WebSocket Subscription Cleanup (Ghost Candles & Daily Stats Fix)
- Implemented a local `activeSubscriptions` Map in [chartInit.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/chartInit.ts) to track the exact subscription parameters (exchange, fileCode) associated with a symbol ticker and timeframe.
- Updated `unsubscribeBar` inside `createDataLoader` to retrieve the correct parameters from `activeSubscriptions` and unsubscribe the event handler from `eventBus` and the feed from `wsClient`, resolving the timing mismatch where active selections updated before unsubscribe events occurred.
- Modified the real-time quote tick processor inside `subscribeBar` to calculate timeframe-specific candle bounds incrementally from the chart's current candles (`chart.getDataList()`) instead of mapping open/high/low to the broker's daily session statistics.
- Added a `mode:change` listener in [chartInit.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/chart/chartInit.ts) to cleanly unsubscribe event handlers and clear local state maps when switching to offline mode.
- Modified `wsClient.disconnect()` in [wsClient.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/openalgo/wsClient.ts) to clear the active subscriptions map, preventing stale subscriptions from reconnecting when returning online.

### Search Modal UI & Focus
- Fixed focus and selection cursor loss in the Symbol Search Modal and Indicator Modal by adding focus and text selection preservation logic across state re-renders in [BaseComponent](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/BaseComponent.ts).
- Added the search icon to the SVG sprite sheet in [icons.svg](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/public/icons.svg) and registered `'search'` in [icons.ts](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/icons.ts).
- Redesigned the search input in the [SymbolSearchModal](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/SymbolSearchModal.ts) and [IndicatorModal](file:///d:/Devs/codetest-og/klinecharts-v10.0.0-beta3/src/ui/IndicatorModal.ts) to be a modern floating search bar with rounded corners, a subtle background, active focus transitions, and highlight effects.

## 2026-06-15

### Symbol Search & Chart Fixes
- Added a timezone offset (`IST_OFFSET_SECONDS = 19800`) to incoming timestamps from the OpenAlgo REST API `/history` endpoint in online mode to correctly display candles in Indian Standard Time (IST).
- Refactored `getFetchDurationDays` to implement dynamic historical lookback range rules (e.g. 10 years for weekly/monthly, 2 years for daily, 180 days for hourly, and 15 or 90 days for minute intervals).
- Implemented `safeParseFloat` utility to prevent NaN propagation on history OHLCV values, and filtered out non-finite candle data, sorted candles, and removed duplicates.
- Refactored `SymbolSearchModal` to map broker search response fields robustly and sorted matches prioritizing exact matches, starts-with prefixes, Nifty 50 matches, and shorter symbol lengths.
- Added interactive filter tabs (All, Stocks, Futures, Options, Indices) to both online and offline symbol search modes, and styled the tabs using variables from the white theme.

### Online Mode
- Added a new "OpenAlgo" tab to the main Settings modal for configuring the Host URL, API Key (with show/hide eye toggle), WebSocket URL, and OpenAlgo Username.
- Persisted OpenAlgo connection parameters directly to `localStorage` on-the-fly.
- Persisted Online/Offline Mode selection in `localStorage` under `openchart_mode` and updated TopBar and Symbol Search Modal to initialize from it.
- Connected Symbol Search to OpenAlgo REST API `/search` in Online Mode, debouncing input queries by 300ms.
- Connected Chart historical data loading to OpenAlgo REST API `/history` in Online Mode, mapping timeframe codes to OpenAlgo interval strings, fetching date pages dynamically on chart scroll, and caching/merging bars.
- Enabled all standard timeframe intervals in online mode.
- Handled `mode:change` event in `ChartManager` to reset chart data and force-reload when switching modes.

### Volume Cluster
- Updated the default Volume Cluster marker colors to `#00E5FF` for bull markers and `#FFB300` for bear markers through the existing customizable color settings.
