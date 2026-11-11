# Cryptween - Trading Intelligence Network

A decentralized trading intelligence platform built with Electron, React, and TypeScript. Cryptween creates a collective layer of trading data through voluntary position sharing, where access is proportional to contribution.

**Current version:** 0.2.0 (beta) — see [CHANGELOG.md](CHANGELOG.md).

## 🌟 Overview

Cryptween is a network built from real positions shared by traders. What you see depends on what people choose to share. Your portfolio remains private by default—you control exactly what to contribute to the collective intelligence.

## 🎨 Design System

**Brand Identity**: "Structured intelligence + live signal"

**Theme Support:**
- **Dark Theme** (default): Dark backgrounds with green/blue accents
- **Light Theme**: Light backgrounds with adjusted contrast
- Seamless theme switching with CSS variables

**Colors:**
- Primary Green (#3ECF8E): Action, participation, signals, CTAs
- Primary Blue (#3A4A6B): Structure, trust, headers
- Background: `var(--color-bg-base)` - Theme-aware
- Card Surface: `var(--color-surface-card)` - Theme-aware

**Typography:** Inter font family
**Shape:** Buttons 12px radius, Cards 16px radius
**Motion:** 200-300ms transitions, green glow for activity, blue tint for structure

## 💡 Core Concepts

### 1. **Privacy by Default**
Your portfolio is completely private. No email, no personal account required. You generate a self-sovereign identity and choose exactly what to share.

### 2. **Proportional Access**
Access to collective intelligence is tied to your contribution:
- **Low Access** (< 40% shared): Partial signal visibility
- **Medium Access** (40-70% shared): Expanded collective data
- **Full Access** (> 70% shared): Complete collective intelligence

### 3. **Data as Signal, Not Profile**
Shared data is aggregated into the collective. It's not tied to your personal identity, and other users cannot track your individual positions. Your data is never sold.

### 4. **Collective Intelligence**
The system is built from real trading positions. The more traders participate, the stronger the signal becomes.

## 🚀 Features

### Portfolio Management
- **Real-time Dashboard**: Track balance, equity, UPL, and delta BTC
- **Multi-Exchange Support**: Aggregate positions across exchanges
- **Tracking Controls**: Start/Stop Tracking in the Portfolio header; the key button next to it opens the exchange accounts panel (only while tracking is stopped). Start is enabled once at least one exchange account is saved
- **Clean Stop**: Stopping tracking clears the portfolio data and stops sharing, so a restart never shows stale values or resumes an old selection
- **Performance Charts**: Total portfolio value with a 15m / 30m / 1h / 3h / 6h window (6 hours of history are kept)
- **Position Tracking**: Detailed breakdown by symbol, type, and exchange
- **Mock Portfolio** (`MOCK_PORTFOLIO_OPTION`): simulated portfolio that enters, increases, reduces, closes and switches positions over time, for testing without exchange accounts

### Identity & Access
- **Self-Generated Identity**: Wallet-style addresses (e.g., 0xA3F...91K)
- **No Email Required**: Privacy-first authentication
- **Credential Management**: Secure storage with password protection
- **Access Levels**: Dynamic badges with hover tooltips explaining contribution requirements

### Sharing System
- **Selective Sharing**: Choose individual positions to contribute, or share all positions
- **Dynamic Filtering**: Share by exchange, symbol, pattern, and position type
- **Minimum Value**: Positions at or below `MIN_SHARING_POSITION_USD` (default $10) are not shared, including positions that shrink below it
- **Beta Curation**: In beta, sharing is limited to a curated set of positions
- **Real-Time Sync**: Daemon-powered background updates with the sharing server
- **Contribution Metrics**: See your contribution baseline and percentage of the collective

### Collective Layer
- **Summary Comparison**: View your contribution vs collective totals
- **Aggregate Data**: Access shared position aggregates
- **Access Visualization**: Clear badges showing your current access level
- **Contribution Insights**: Understand your participation in the network

### Market Structure & Intelligence Map
- **Market Structure Window**: Per-symbol market share of the shared positions, recomputed as soon as the shared set changes (see [MARKET_STRUCTURE_WINDOW.md](docs/MARKET_STRUCTURE_WINDOW.md))
- **Intelligence Map**: One bar per shared room, coloured by its MAV (Market Awareness Vector) reading, tagged with instrument (SPOT, FUT, PERP, OPT…) and pattern
- **Metrics Panel**: Click a bar, the origin or an exchange to pin it; sentiment and trading risk come from the sharing server
- **Indicator Plot Window**: Per-room indicator chart

### Following System
- **Follow Traders**: Monitor other network participants
- **Summary Tracking**: View balance, equity, UPL, and delta BTC for followed traders
- **Beta Notice**: Following is open in beta - full version will require user permission

### Trading Rooms
- **Chat-Server Backed**: Rooms are chat server conversations joined with the daemon's room metrics
- **Categories**: Trading rooms under Cross-Exchange, Exchange Summaries and Positions; normal conversations under My Rooms (owned) and Cross Rooms
- **Room Metrics**: Participants, sentiment and MAV refreshed every 5 seconds
- **Only Updated Rooms**: A trading room is listed only once the sharing server has returned its aggregate or summary, so rooms never show 0 participants and no metrics
- **Stale Rooms**: A room the server fails to update keeps its last metrics; it is flagged "not updated" only after `TRADING_ROOM_STALE_AFTER_ERRORS` failed updates in a row (default 3), so a momentary network failure is not reported
- **Message Sentiment**: Per-message positive / negative / neutral scores
- **Privacy**: A trader's full blockchain address is never shown (alias or shortened login)
- **Access-Gated**: "Launch Trading Rooms" is enabled once at least one of your positions is shared
- **Separate Window**: Runs in a dedicated Electron window, with logout in the rooms header

### Chat Integration
- **Built-in Messaging**: Communicate within the platform
- **Membership Sync**: Rooms you were removed from disappear and rooms you were added to appear without a manual reload
- **Connection Recovery**: Bounded reconnect attempts; message polling is used only while disconnected

## 🔧 Architecture

### Daemon Integration
The app integrates with a background daemon service that handles:
- Position data aggregation
- Real-time summary updates
- Collective intelligence computation
- Follow user tracking
- Server synchronization
- Non-overlapping asynchronous polling and bounded histories/caches
- Exchange HTTP configuration and rate limiting through a shared client base
- Validation of exchange response shapes before normalization
- HTTPS for production sharing and identity-provider endpoints

### State Management
- **3 User States**: OUTSIDE, INSIDE_NOT_SHARING, SHARING
- **Context API**: Global state for user, positions, and access levels
- **Custom Hooks**: `useIdentity`, `useSharing`, `useTimedDisplay`
- **Secure Persistence**: Electron `safeStorage` for persisted sessions and exchange credentials

### Multi-Window System
- **Main Window** (`index.html`, `src/renderer`): Primary app interface
- **Trading Rooms Window** (`chat.html`, `src/renderer_chat`): Trading rooms and chat
- **Market Structure Window** (`market.html`, `src/renderer_market`)
- **Indicator Plot Window** (`plot.html`, `src/renderer_plot`)
- **IPC Communication**: Electron bridge for window coordination, with runtime validation of payloads (`ipcValidation.ts`)
- **Renderer Recovery**: Shared React Error Boundary in all four renderer entry points
- **Logging**: Shared logger for main and renderers — console in development, errors only to a rotated `main.log` in the packaged app

## ⚠️ Beta Version Notes

- The beta build connects to the hosted sharing server, identity provider and chat server by default
- Portfolio data can be mocked (`MOCK_PORTFOLIO_OPTION`) for testing
- Beta sharing is limited to a curated set of positions
- Identity is not yet backed by a real wallet
- Following is open without permission - full version will require consent

## 📁 Project Structure

```
cryptween-beta/
├── .github/workflows/build.yml  # Test, build and release workflow
├── src/
│   ├── main/                    # Electron main process
│   │   ├── main.ts              # Windows & IPC handlers
│   │   ├── preload.ts           # Context bridge
│   │   ├── config.ts            # Dev / beta / production configuration
│   │   ├── daemon.ts            # Background service (portfolio, sharing, metrics)
│   │   ├── ipcValidation.ts     # Runtime validation of IPC payloads
│   │   ├── logger.ts            # Main-process logger (rotated main.log)
│   │   ├── credentialsStore.ts  # safeStorage-backed exchange credentials
│   │   ├── authTokenStore.ts    # safeStorage-backed session
│   │   ├── appUpdateService.ts  # Verified auto-update
│   │   ├── exchanges/           # Exchange API clients (BaseExchangeClient)
│   │   ├── marketStructure/     # Market structure engines
│   │   ├── porfolio-tracker/    # Real portfolio tracker
│   │   └── __tests__/           # Vitest suites
│   ├── renderer/                # Main React application
│   │   ├── components/          # Reusable UI components
│   │   │   ├── Button.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Banner.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── TopBar.tsx
│   │   │   ├── BottomNav.tsx
│   │   │   ├── AccessBadge.tsx  # With hover tooltips
│   │   │   ├── AnimatedWallet.tsx
│   │   │   ├── Onboarding.tsx   # Welcome slides
│   │   │   ├── PositionCheckbox.tsx
│   │   │   ├── NetworkAnimation.tsx
│   │   │   ├── CredentialModal.tsx
│   │   │   ├── AddFollowUserModal.tsx
│   │   │   └── DebugPanel.tsx
│   │   ├── screens/             # Screen components
│   │   │   ├── Portfolio.tsx
│   │   │   ├── LockedState.tsx
│   │   │   ├── InsideNotSharing.tsx
│   │   │   ├── SelectPositions.tsx
│   │   │   ├── SelectAggregateFilters.tsx
│   │   │   ├── SharingActivation.tsx
│   │   │   ├── CollectiveLayer.tsx
│   │   │   ├── FollowingEmpty.tsx
│   │   │   ├── FollowingActive.tsx
│   │   │   └── RoomsLauncher.tsx
│   │   ├── context/
│   │   │   └── AppContext.tsx   # Global state management
│   │   ├── hooks/
│   │   │   ├── useTimedDisplay.ts
│   │   │   ├── useIdentity.ts
│   │   │   └── useSharing.ts
│   │   ├── types/
│   │   │   └── index.ts         # TypeScript interfaces & enums
│   │   ├── utils/
│   │   │   ├── mockData.ts
│   │   │   ├── accessLevel.ts   # Access level calculations
│   │   │   ├── messages.ts      # UX flow messaging
│   │   │   ├── analytics.ts     # Event tracking
│   │   │   └── externalModules.ts
│   │   ├── styles/
│   │   │   ├── variables.css    # Design tokens (theme-aware)
│   │   │   ├── global.css       # Base styles
│   │   │   └── animations.css   # Transitions & effects
│   │   ├── layouts/
│   │   │   └── MainLayout.tsx
│   │   ├── App.tsx
│   │   └── index.tsx
│   ├── renderer_chat/           # Trading Rooms / chat window
│   │   ├── api/                 # Chat server WebSocket client
│   │   ├── providers/tradingRooms/  # Rooms list (chat conversations + daemon metrics)
│   │   ├── components/
│   │   ├── services/
│   │   ├── store/               # Redux store
│   │   └── index.tsx
│   ├── renderer_market/         # Market structure window
│   ├── renderer_plot/           # Indicator plot window
│   └── shared/                  # Shared components and renderer logger
├── images/                      # App images
├── logos/                       # App icons
├── docs/                        # Documentation
├── scripts/                     # Development scripts
├── index.html                   # Main window entry
├── chat.html                    # Trading Rooms window entry
├── market.html                  # Market structure window entry
├── plot.html                    # Indicator plot window entry
├── .env.example                 # Environment variable template
├── package.json
├── tsconfig.json
├── vite.config.mts
├── vitest.config.ts
└── README.md
```

## 🛠️ Development

### Prerequisites
- Node.js 22 (same version as CI)
- npm 10+

### Installation
```bash
npm install
cp .env.example .env   # then adjust the values (see Configuration)
```

### Available Scripts

#### Development
```bash
npm run dev            # Vite dev server on http://localhost:3000 + Electron with hot reload
npm run dev:instance2  # Second Electron instance with its own profile (./.dev-profile-2)
npm run clean          # Remove build output, caches and the Electron userData folder
```

#### Testing
```bash
npm test               # Run the Vitest suite once
npm run test:watch     # Watch mode
npm run test:coverage  # Coverage report (all files under src)
```

### TradingRooms Data Sources

TradingRooms combines two sources:

- Conversations, messages and participants come from the Sama chat server (`@services`).
- Room metrics come from the daemon over IPC (`window.electronAPI.daemon.getTradingRooms`).
  A daemon room is linked to the conversation whose name matches its
  `cryptween.aggregates.positions.*` / `cryptween.aggregates.summaries.*` key
  (see `src/renderer_chat/providers/tradingRooms/realTradingRoomsProvider.ts`).

#### Building
```bash
npm run electron:build # Typecheck (tsc) and build renderers + Electron main process
```

#### Packaging
```bash
npm run package:portable:win    # release/Cryptween-win-unpacked-<version>.zip
npm run package:portable:mac    # release/Cryptween-mac-portable-<version>.zip (universal)
npm run package:portable:linux  # release/Cryptween-linux-portable-<version>.tar.gz
npm run package                 # Installers (NSIS / DMG / AppImage) with electron-builder
```

All packages use `logos/tt.png` as the icon and bundle the root `.env` as the app's runtime configuration.

#### Automated Builds & Releases

The workflow [.github/workflows/build.yml](.github/workflows/build.yml) runs on `v*` tag pushes and on manual dispatch:

1. **test** — `npm ci` + `npm test` on Ubuntu. A failing test stops the workflow.
2. **build** — on Ubuntu, Windows and macOS: generates the `.env` from secrets and variables, verifies the chat endpoints are not local, builds and packages the portable binaries.
3. **release** (tags only) — publishes the binaries and `checksums.txt` to `cryptween/cryptween-binaries`. Tags containing `alpha`, `beta` or `rc` are marked as pre-release.

To release a new version:
```bash
npm version minor --no-git-tag-version   # or patch / major
git commit -am "chore: release v0.2.0"
git tag v0.2.0
git push origin master --tags
```

**Repository secrets** (Settings › Secrets and variables › Actions › Secrets):

| Secret | Required | Purpose |
|---|---|---|
| `JWT_SECRET` | Yes | Signs access tokens; must match the sharing server / IDP. The build fails without it. |
| `BINARIES_REPO_TOKEN` | Yes (release) | PAT with Contents: write on `cryptween/cryptween-binaries`. |
| `BETA_VERSION_SECRETS` | Recommended | Beta mode secret (falls back to `default_secret`). |
| `APP_EVALUATION_TOKEN` | Optional | Enables in-app evaluation; empty disables it. |

**Repository variables** (… › Variables), all optional — defaults shown:

| Variable | Default |
|---|---|
| `API_BASE_URL` | `https://sharing.cryptween.app` |
| `IDP_BASE_URL` | `https://idp.cryptween.app` |
| `SIM_USE_SHARING_SERVER` | `true` |
| `SIM_USE_IDP` | `true` |
| `TRADING_ROOM_STALE_AFTER_ERRORS` | `3` |
| `VITE_SOCKET_CONNECT` | `wss://chat.cryptween.app` |
| `VITE_HTTP_CONNECT` | `https://chat.cryptween.app` |
| `VITE_ORGANIZATION_ID` | `65f1c2a0e7b4a1d3c9f2ab10` |

See [BUILD_AND_RELEASE.md](docs/BUILD_AND_RELEASE.md) for code signing and troubleshooting (including running on Linux as a non-root user).

### Configuration Files

- **vite.config.mts**: Vite bundler configuration (four HTML entry points + Electron main/preload)
- **vitest.config.ts**: Test runner configuration
- **tsconfig.json**: TypeScript compiler config for renderers
- **tsconfig.node.json**: TypeScript config for Electron main process
- **tsconfig.scripts.json**: Strict TypeScript config for `scripts/`
- **package.json**: Dependencies and build scripts

### Development Tips

1. **Hot Reload**: Changes to renderer code auto-reload
2. **Main Process**: Changes to main.ts require restart
3. **DevTools**: Open automatically in development mode
4. **Theme Testing**: Toggle between light/dark themes in TopBar
5. **State Debugging**: Use DebugPanel component (Shift+D to toggle)

## 🎯 User Journey

### First Launch & Onboarding

1. **App Opens**: Portfolio screen displays immediately
2. **Welcome Slides** (4 slides):
   - **Slide 1**: "A network built from real positions"
   - **Slide 2**: "You stay in control" - Privacy and control explained
   - **Slide 3**: "Your data becomes signal" - Data aggregation explained
   - **Slide 4**: "This is an early version" - Beta clarifications
   - Use **Previous/Next** buttons to navigate
   - Click **Skip** (first slide) or **Done** (last slide) to proceed

3. **Timed Hints**:
   - After 1s: "Cryptween is a network built from shared positions"
   - After 2.5s: Locked tabs (Collective, Following, Rooms) pulse

### Entering the Network

1. **Access Locked Feature**: Click Collective, Following, or Rooms tab
2. **See Context Message**: Tab-specific explanation with beta footnote
3. **Enter Cryptween**:
   - Click "Enter Cryptween" button
   - Enter credentials (address, password, access secret)
   - System validates and generates identity
4. **Identity Confirmation**:
   - Animated wallet address generation (1-1.5s)
   - Format: `0xA3F...91K`
   - Click "Enter the network" to proceed

### Inside the Network (Not Sharing)

1. **All Tabs Unlock**: Can view all sections
2. **Timed Messages**:
   - After 1s: Banner "You're inside the network, but not sharing yet"
   - After 2s: "Begin sharing" CTA appears
3. **Limited Access**: Can see interface but not collective data

### Starting to Share

1. **Click "Begin sharing"**: Opens position selection screen
2. **Select Positions**:
   - Filter by exchange, symbol, pattern, or type
   - Check individual positions or use bulk actions
   - Real-time counter: "Sharing X of Y positions"
3. **See Proportionality Hints**:
   - After 1.5s: "More positions shared → greater access"
   - Preview your potential access level
4. **Begin Sharing**:
   - Click "Begin sharing" button
   - Activation animation with green network flow
   - Message: "You are now sharing your selected positions"

### Using the Collective Layer

1. **Access Level Badge**: Shows Low/Medium/Full Access
   - Hover over badge to see explanation and requirements
2. **Your Contribution Summary**: View your baseline metrics
   - Balance, Equity, UPL, Delta BTC, Available Balance (single row)
3. **Collective Comparison**: See your contribution vs collective totals
   - Percentage contribution for each metric
4. **Update Positions**: Modify what you're sharing anytime
5. **Stop Sharing**: Return to not-sharing state (with confirmation)

### Following Traders

1. **Empty State**: "You are not following any traders"
   - Beta note: "Following is open in beta version"
2. **Add Users**:
   - Click "Follow Users"
   - Enter wallet address and nickname
   - Add multiple users
3. **Active Following**:
   - View list of followed traders
   - See their summary metrics (Balance, Equity, UPL, Delta BTC)
   - Manage users (add/remove)
   - Stop following all

### Launching Trading Rooms

1. **Click "Launch Trading Rooms"** in Collective Layer or Rooms tab
2. **New Window Opens**: Separate Electron window
3. **Access Based on Sharing**: Full features proportional to your contribution

### Access Levels Explained

- **Low Access (< 40% shared)**:
  - "You're seeing part of the signal"
  - Hover tooltip: "Share more to unlock more collective signal"
  
- **Medium Access (40-70% shared)**:
  - "Partial access to collective signal"
  - Hover tooltip: Explains current access range
  
- **Full Access (> 70% shared)**:
  - "Full access to collective signal"  
  - Hover tooltip: Confirms complete access

## 🛠️ Tech Stack

### Frontend
- **React 18**: UI library with hooks
- **TypeScript**: Type-safe development
- **Vite**: Fast build tool and dev server
- **CSS Variables**: Theme-aware styling
- **Recharts**: Chart visualization

### Desktop Framework
- **Electron**: Cross-platform desktop apps
- **IPC Communication**: Main-renderer process bridge
- **Multi-Window**: Separate windows for chat and rooms

### State Management
- **React Context API**: Global state (main window)
- **Redux Toolkit**: Trading Rooms / chat window state
- **Custom Hooks**: Reusable logic
- **Electron safeStorage**: OS-backed encryption for persisted sessions and exchange credentials

### Build & Dev Tools
- **electron-builder**: App packaging
- **TypeScript Compiler**: Type checking
- **Vitest**: Unit and regression tests (run in CI before every release)
- **GitHub Actions**: Test, multi-platform build and release

## 🎨 Key UI Components

### Core Components
- **Button**: Primary/Secondary/Danger variants with size options
- **Card**: Themed surface with hover effects
- **Modal**: Centered overlay for dialogs
- **Banner**: Timed messages with animations
- **AccessBadge**: Interactive badges with tooltips

### Specialized Components
- **AnimatedWallet**: Wallet address generation animation
- **NetworkAnimation**: Green flow effect for sharing activation
- **Onboarding**: Multi-slide welcome experience with navigation
- **CredentialModal**: Secure login form
- **AddFollowUserModal**: Follow user management
- **PositionCheckbox**: Enhanced checkbox for position selection

### Layout Components
- **TopBar**: App header with identity and theme switcher
- **BottomNav**: Tab navigation (Portfolio/Collective/Following/Rooms)
- **MainLayout**: Root layout wrapper with content area

## 📝 Configuration

### Environment Variables

Configuration is read once at startup by [src/main/config.ts](src/main/config.ts). Use [.env.example](.env.example) as the template.

- **Development** (`npm run dev`): loads `.env.local` then `.env` (`.env.local` wins, like Vite). Every value comes from the environment, with local defaults (`http://localhost:3535`, `http://localhost:3539`, `ws://localhost:9001`).
- **Packaged app**: loads only the `.env` bundled at build time. `APP_ENV` selects the profile:
  - `beta` (default): hosted servers by default, overridable by the variables below
  - `production` / `prod`: fixed hosted servers, sharing server and IDP always on, beta features off

| Variable | Used by | Purpose |
|---|---|---|
| `APP_ENV` | packaged | `beta` or `production` |
| `JWT_SECRET` | all | Required in packaged builds; dev falls back to an insecure sample value |
| `API_BASE_URL` / `IDP_BASE_URL` | dev, beta | Sharing server and identity provider URLs |
| `SIM_USE_SHARING_SERVER` / `SIM_USE_IDP` | dev, beta | Turn the sharing server / IDP on or off (beta default `true`, dev default `false`) |
| `VITE_SOCKET_CONNECT` / `VITE_HTTP_CONNECT` | all | Chat server endpoints — embedded at build time |
| `VITE_ORGANIZATION_ID` | all | Chat organization — embedded at build time |
| `VITE_MESSAGES_COUNT_TO_PRELOAD` | all | Chat messages loaded per conversation |
| `MOCK_PORTFOLIO_OPTION` | dev, beta | Use the mocked portfolio |
| `MOCK_BEHAVIOUR_INTERVAL_MS` | dev, beta | Interval between mock portfolio behaviour changes |
| `MIN_SHARING_POSITION_USD` | all | Minimum position value for sharing (default 10) |
| `TRADING_ROOM_STALE_AFTER_ERRORS` | all | Consecutive failed sharing-server updates before a trading room is flagged "not updated" (default 3, min 1) |
| `BETA_VERSION` / `BETA_VERSION_SECRETS` | dev, beta | Beta mode flag and secret |
| `APP_EVALUATION_TOKEN` | all | Enables the in-app evaluation |
| `SIM_BLOCKCHAIN_ID`, `SIM_INTERVAL_MS`, `MARKET_STRUCTURE_*`, `MAV_INTERVAL_MS`, `SENTIMENT_INTERVAL_MS`, `RISK_INTERVAL_MS`, `UPLOAD_METRICS_ENABLED` | dev | Tuning of the daemon loops |

> `VITE_*` variables are compiled into the renderer bundle by `vite build`; changing them requires a new build.

### Daemon Configuration
The background daemon handles:
- Real-time position aggregation
- Summary calculations
- Collective intelligence processing
- Network synchronization

User credentials (address, password, access secret) are entered in the credential modal; exchange accounts in the Portfolio accounts panel.

## 🔐 Security Notes

- **Sharing Is Opt-In**: Portfolio data is sent to the configured sharing service only when the user enables sharing; local portfolio and exchange data remain on device otherwise
- **Credential Storage**: Persisted sessions and exchange credentials use Electron `safeStorage`; credentials are not stored in renderer `localStorage` or plaintext portfolio cache files
- **Self-Sovereign Identity**: Addresses are generated locally; beta identities are simulated and are not wallet-backed
- **Update Integrity**: Release downloads are tied to the selected GitHub release asset and checked against its SHA-256 checksum before being saved
- **Checksum Limitation**: The checksum is published alongside the release and does not prove publisher authenticity if the release repository or account is compromised
- **Dependency Audit**: Runtime dependencies currently audit clean; development/build tooling still has advisories that require major-version upgrades

## 🤝 Contributing

This is a beta project. Contributions, issues, and feature requests are welcome.

### Development Workflow
1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

### Code Style
- Use TypeScript for all new code
- Follow existing naming conventions
- Add CSS variables for theme-aware colors
- Use functional React components with hooks
- Keep components focused and single-purpose

## 🐛 Known Issues

- Daemon connection may require restart on first launch
- Theme switching requires window reload for some components
- Following system is open in beta (no permission checks)
- Metrics/research upload is disabled until the sharing server exposes those endpoints

## 📄 License

This project is proprietary software. All rights reserved.

## 📧 Contact

For questions or support, please open an issue in the repository.

---

**Built with ❤️ for the Cryptween community**

## 👥 Credits

Built with:
- Electron 28
- React 18
- TypeScript 5
- Vite 5
- Vitest 4
- Recharts 2
- Inter Font Family

---

**Status**: 0.2.0 beta — connected to the hosted sharing server, identity provider and chat server, with tests gating every release.
