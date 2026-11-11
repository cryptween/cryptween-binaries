# Changelog

All notable changes to the Cryptween project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.2.0] - 2026-11-10

### Added
- **Portfolio tracking controls**: Start/Stop Tracking button in the Portfolio header, next to a key button that opens the exchange accounts panel
- **Portfolio value chart**: selectable time window (15m / 30m / 1h / 3h / 6h, default 1h)
- **Share all positions** option on the position selection screen
- **Trading Rooms categories**: conversations are classified by the chat server's `cryptween_type`; normal conversations go to "My Rooms" (owned) or "Cross Rooms", trading rooms stay under Cross-Exchange, Exchange Summaries and Positions
- Trading rooms the sharing server fails to update keep their last metrics, and are flagged "not updated" only after `TRADING_ROOM_STALE_AFTER_ERRORS` consecutive failed updates (default 3), so a momentary network failure is not reported. Each shared aggregate/summary tracks its consecutive errors (`errorCount`, `errorSince`), reset by the next good update
- Intelligence map bar tags show the instrument (SPOT, FUT, PERP, OPT…) next to the pattern, and the legend lists the abbreviations
- Mock portfolio simulates user behaviour (enter / increase / reduce / close / switch spot ↔ future) every `MOCK_BEHAVIOUR_INTERVAL_MS` (default 10 min)
- Shared logger for the main process and all renderers: console in development, errors only (rotated `main.log`) in the packaged app
- Runtime validation of IPC payloads (`ipcValidation.ts`); unknown exchanges are rejected
- Identity-provider connection errors are now shown to the user
- Per-message sentiment scores (positive / negative / neutral) in Trading Rooms chat bubbles
- Logout button in the Trading Rooms window, with a signed-out screen and "Sign in again"
- Trading room metrics refresh every 5 seconds, so the rooms window stays in sync with the daemon
- @lynah messages carry the room's current metrics (sentiment, and MAV for position rooms); new timeframe tags (`@last_N_*`, `@range_*`, `@date_*`)
- Sentiment and trading risk polled from their own sharing server endpoints, plus global MAV readings for summary rooms
- Beta sharing restricted to a curated set of positions, with a one-time notice in the Collective layer
- Minimum position value for sharing (`MIN_SHARING_POSITION_USD`, default $10), with a notice on the position selection screen
- Click a bar, the origin or an exchange on the MAV map to pin it to the metrics panel
- Comprehensive README.md with full documentation
- Shared React Error Boundary with a reload fallback across all four renderers
- Welcome image background for InsideNotSharing screen
- Previous button in onboarding slides for backward navigation
- Information tooltips on AccessBadge with hover explanations
- Balance metric to Collective Layer contribution summaries
- Beta clarification messages for Following feature across all screens
- Footnote support in LockedState component for contextual information

### Changed
- **Beta builds use the hosted servers by default**: the packaged beta now connects to the sharing server and the identity provider (`SIM_USE_SHARING_SERVER` / `SIM_USE_IDP`, default `true`), and `API_BASE_URL` / `IDP_BASE_URL` can be overridden from the packaged `.env`
- Trading rooms (`cryptween_type` 2) are listed only when the daemon has an updated aggregate or summary for them: a room the sharing server never answered (e.g. its first request failed) is not listed, instead of showing 0 participants and no metrics
- Stopping portfolio tracking also stops sharing and clears the position selection; the user chooses positions again after restarting tracking
- Stopping portfolio tracking clears summaries, positions, aggregates and history, so a restart never shows stale values
- The exchange accounts panel opens only while tracking is stopped; "Save & Connect" became "Save" and no longer starts tracking
- Sharing requires at least one position: "Share all positions" is disabled without positions, and "Launch Trading Rooms" is disabled until a position is shared
- The minimum position value is applied when sending, so positions that shrink below it stop being shared
- Market structure recomputes as soon as the shared symbol/type set changes, prunes pairs that stopped being shared, and isolates each product engine
- Intelligence map: bars are spaced evenly and thinned as they multiply, coloured by MAV only (grey without a reading); the summary legend section is now "Portfolio Origin"
- The MAV chart no longer moves with the mouse wheel or drag
- Trading rooms never show a trader's full blockchain address (alias or shortened login instead)
- Logout and New Room are icon buttons in the rooms list header
- Daemon summary history keeps the last 6 hours instead of the last 1000 entries
- In development the main process loads `.env.local` before `.env`, matching Vite
- Metrics/research upload is off by default (`UPLOAD_METRICS_ENABLED`), and queued metric events are capped at 1000
- "Your Contribution" is now the USD value of your shared positions over the collective value of the same rooms
- Capital flow direction comes from MAV D_flow; netFlow and simulated risk values were removed
- Portfolio Max Loss Span is shown as a dollar amount instead of a percentage
- The MAV awareness map is always shown on a bar's card, placed at (0, 0) until the server has a reading
- Production sharing and identity-provider endpoints now use HTTPS
- Chat message fallback polling runs only while the WebSocket is disconnected
- Revealed exchange secrets automatically hide after 15 seconds
- Daemon update loops now skip overlapping cycles; retained room and market data caches have capacity limits and evict stale entries
- Exchange API clients now share HTTP setup and rate-limit handling through `BaseExchangeClient`
- WebSocket reconnects now use bounded exponential backoff with jitter, connection/request timeouts, and cleanup of pending requests and listeners
- Script TypeScript configuration now enables strict checking
- Updated onboarding slide content with clearer messaging:
  - Slide 1: "A network built from real positions"
  - Slide 2: Enhanced privacy explanation
  - Slide 3: Data as signal concept
  - Slide 4: Beta version clarifications with simulated data notes
- Improved navigation buttons to text-only style (removed button borders)
- Following feature clarification: requires "other user permission" in full product
- Updated heading: "Your Contribution Baseline" → "Your Contribution Summary (Baseline)"
- Updated heading: "Collective vs Your Contribution" → "Collective Summary vs Your Contribution"
- Reduced metric box sizes in Collective Layer for single-row display (5 metrics)
- Chart and UI colors now use CSS variables for theme support

### Removed
- Mock trading rooms flow (`MOCK_TRADING_ROOMS_OPTION`, the `mock=true` window flag) and the daemon's in-memory chat rooms; trading rooms always run against the chat server

### Fixed
- Late server responses that arrive after sharing stopped no longer refill the shared layer
- Positions load in the Portfolio layer when the portfolio is mocked
- Trading Rooms window can be opened again when the user is not sharing positions
- Shared aggregates returned with an error field now appear in the Trading Rooms list
- Rooms the user was silently removed from (auto-removal) are dropped from the list; rooms the user was added to appear without a manual reload
- A message refused because the user left the room now says so instead of "check your connection"
- Market structure SPOT share used the previous snapshot total (0% on the first run)
- Mock portfolio position sizes are bounded, so behaviour changes can't compound into unrealistic totals
- Metric values are validated when the server returns nulls
- Logout clears the saved session only after the server-side logouts run, and no longer hangs on a dead socket
- MAV, sentiment and risk polling skip aggregates with no type/pattern/symbol (no more `///` 404 requests)
- Script symbol-fetch helper now returns successful responses and safely handles unknown errors
- Async portfolio, metrics, MAV, sentiment, and risk loops now catch failures and skip overlapping runs
- Exchange response parsers validate expected arrays, envelopes, row lengths, and finite numeric values before indexing
- Chat typing failures and fallback polling errors now emit diagnostics instead of being silently discarded
- Theme support: Removed hardcoded dark colors preventing light theme display
- LockedState component now properly supports theme switching
- Portfolio chart (Recharts) now uses theme-aware colors
- Chart tooltip colors adapt to current theme

### Security
- Auto-update downloads are bound to assets returned by the official release check, restricted to approved HTTPS hosts, size-limited, and SHA-256 verified before the temporary file is moved to its destination
- Persisted authentication sessions and exchange credentials use Electron `safeStorage` instead of plaintext renderer storage or portfolio cache data
- Production API and identity-provider traffic uses HTTPS
- Release checksums detect corruption but are not a substitute for a signed release manifest

### Technical
- **CI**: the release workflow runs the test suite (`npm test`) first; a failing test blocks the build and the release
- **CI**: the generated `.env` includes the server URLs, the sharing/IDP switches and the chat endpoints (`VITE_SOCKET_CONNECT`, `VITE_HTTP_CONNECT`, `VITE_ORGANIZATION_ID`), each overridable with a repository variable; the build fails if a chat endpoint is missing or points to localhost
- `*.tsbuildinfo` is no longer tracked
- Added focused regression tests for update integrity, daemon loops/cache eviction, WebSocket resilience, and the shared exchange client
- Vitest coverage now includes all TypeScript/TSX files under `src`
- Updated compatible dependency patches, including Axios, form-data, Moment, React Router, and Monaco/DOMPurify
- Runtime dependency audit is clear; remaining development/build-tool advisories require major-version migration

### Platform Notes
- Linux releases must run as a regular, non-root user so Chromium's sandbox can operate; do not launch with `--no-sandbox`. See [BUILD_AND_RELEASE.md](docs/BUILD_AND_RELEASE.md#app-refuses-to-start-as-root-on-linux).

## [0.1.1] - 2024

### Added
- **Onboarding System**: Multi-slide welcome experience with 4 informative slides
  - Navigation with Previous/Next buttons
  - Skip functionality on first slide
  - Beta context and data handling explanations
- **Theme Support**: Light and dark theme with seamless switching
  - CSS variables for all colors
  - Theme-aware components throughout the app
  - TopBar theme toggle
- **Access Level System**: Three-tier proportional access
  - Low Access (< 40%): Partial signal visibility
  - Medium Access (40-70%): Expanded data access
  - Full Access (> 70%): Complete collective intelligence
  - Interactive badges with hover tooltips explaining requirements
- **Following System**: Track other traders in the network
  - Add/remove followed users with nicknames
  - View real-time summary metrics for followed traders
  - Empty and active states with smooth transitions
  - Beta notice about permission requirements
- **Collective Layer Enhancements**:
  - Contribution baseline metrics (Balance, Equity, UPL, Delta BTC, Available Balance)
  - Comparison view: Your contribution vs collective totals
  - Percentage contribution calculations
  - Compact single-row metric display
  - Access level insights and recommendations
- **Daemon Integration**: Background service for data aggregation
  - Real-time position synchronization
  - Summary calculation and caching
  - Follow user tracking
  - Server communication layer
- **Credential System**: Secure authentication with password protection
  - Credential modal for login
  - LocalStorage caching for convenience
  - Access secret support
  - Wallet address validation

### Changed
- **Design System**: Refined brand identity
  - Primary Green (#3ECF8E) for actions and signals
  - Primary Blue (#3A4A6B) for structure and trust
  - Theme-aware backgrounds and surfaces
  - Inter font family throughout
- **State Management**: Improved global state with Context API
  - Three user states: OUTSIDE, INSIDE_NOT_SHARING, SHARING
  - Dynamic access level calculation
  - Persistent identity management
- **Position Selection**: Enhanced filtering and selection
  - Filter by exchange, symbol, pattern, and type
  - Bulk select/deselect actions
  - Real-time counter updates
  - Proportionality hints
- **Navigation**: Bottom tab system with lock states
  - Portfolio (always accessible)
  - Collective (gated by sharing)
  - Following (gated by network entry)
  - Rooms (gated by sharing)

### Technical
- **Multi-Window Architecture**: 
  - Main window (renderer)
  - Chat window (renderer_chat)
  - Trading Rooms (external module)
- **IPC Communication**: Electron bridge for cross-window messaging
- **Build System**:
  - Vite for fast development and building
  - TypeScript for type safety
  - electron-builder for packaging
- **Component Library**:
  - 15+ reusable UI components
  - 10+ screen components
  - Custom hooks for timing and state
  - Animated components for UX flow

## [0.1.0] - Initial Beta Release

### Added
- **Portfolio Management**: Private portfolio tracker
  - Real-time balance and equity tracking
  - Performance charts with Recharts
  - Multi-exchange position aggregation
  - Delta BTC calculations
- **Identity System**: Self-sovereign identity generation
  - Wallet-style addresses (0xA3F...91K format)
  - Animated wallet generation
  - No email or personal account required
- **Sharing Mechanics**: Voluntary position contribution
  - Select individual positions to share
  - Dynamic sharing percentage calculation
  - Access level tied to contribution
- **Collective Layer**: Aggregated trading intelligence
  - Shared position visualization
  - Collective summary metrics
  - Access-based data visibility
- **Trading Rooms**: External integration launcher
  - Separate Electron window
  - Access gated by sharing level
- **UX Flow**: Complete 9-phase user journey
  - Phase 1: App launch with timed hints
  - Phase 2: Locked state with contextual messaging
  - Phase 3: Identity creation with animation
  - Phase 4: Inside network (not sharing)
  - Phase 5: Position selection
  - Phase 6: Sharing activation with animation
  - Phase 7: Collective layer access
  - Phase 8: Trading rooms launcher
  - Phase 9: Following system

### Design
- Dark-themed UI with green and blue accents
- Smooth animations and transitions (200-300ms)
- Green glow effects for activity
- Blue tint for structural elements
- Card-based layout with 16px radius
- Button components with 12px radius

### Technical Stack
- **Frontend**: React 18 + TypeScript
- **Desktop**: Electron with IPC
- **Build**: Vite + electron-builder
- **Styling**: CSS with design tokens
- **Charts**: Recharts library
- **State**: Context API with custom hooks

---

## Release Notes Format

### Version Number Guidelines
- **Major** (X.0.0): Breaking changes or major feature releases
- **Minor** (0.X.0): New features, backwards compatible
- **Patch** (0.0.X): Bug fixes and minor improvements

### Change Categories
- **Added**: New features
- **Changed**: Changes to existing functionality
- **Deprecated**: Soon-to-be removed features
- **Removed**: Removed features
- **Fixed**: Bug fixes
- **Security**: Security improvements
- **Technical**: Internal technical changes
- **Design**: UI/UX improvements

---

**Note**: This is a beta version. Since 0.2.0 the beta build connects to the hosted sharing server, identity provider and chat server by default. Portfolio data can still be mocked (`MOCK_PORTFOLIO_OPTION`), beta sharing is limited to a curated set of positions, and identity is not yet backed by a real wallet.
