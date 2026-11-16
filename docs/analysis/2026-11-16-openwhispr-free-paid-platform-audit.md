# OpenWhispr: free vs. paid, Windows, and Android sync audit

Date: 2026-11-16
Fork: `chirag127/openwhispr` at `repos/fork/openwhispr`
Upstream: `OpenWhispr/openwhispr`
Audited commits: desktop `2eb67aa55` (v1.11.1), upstream after fetch `e6834b20d`; mobile v1.1.0
License: MIT

## Executive summary

- **The desktop app already gives every on-device feature away for free.** Local transcription, local language models, local diarization, local notes/search, the custom dictionary, and BYOK all work without a paid plan. No local desktop feature is gated behind a subscription.
- **The paid items are cloud services with one exception: connectors.** Hosted transcription above the free allowance, cross-device sync, public note sharing, hosted API/MCP, hosted CLI login, team creation, and meeting hours are all server-backed and therefore reasonable paid features. Connectors, however, authenticate directly to GitHub/Gmail/Slack/Linear from the desktop and call those APIs directly; there is no OpenWhispr-hosted cost to recover, so the Pro gate is artificial.
- **The free/paid story is not documented.** The README says "free" and MIT but never states the Cloud allowance or paid plans. The in-app pricing copy also lists "Mobile app" as a Pro feature even though the mobile app and its BYOK mode are free.
- **Android is not at parity with iOS.** Android has on-device Whisper, but no BYOK provider setup, no Parakeet, no on-device text reasoning (Apple Intelligence is iOS-only), no iOS-style keyboard (it uses a floating bubble), and no Google Calendar integration. Mobile CI validates an iOS bundle only.
- **Windows has no Windows-specific paywall.** The Windows build uses the same free local features. The remaining Windows problems are bugs/quality gaps (elevated windows, long-paste reliability, microphone rate, hit targets), not monetization.

## 1. Free vs. paid matrix (desktop)

Derived from the in-app pricing strings in `src/locales/en/translation.json:2504-2559` and the gates in `src/hooks/useUsage.ts`, `src/lib/capabilities.ts`, and `src/utils/connectorEligibility.ts` (now removed by PR #3258).

| Capability                                               | Free               | Paid                                     | Notes                                  |
| -------------------------------------------------------- | ------------------ | ---------------------------------------- | -------------------------------------- |
| Local transcription (Whisper, Parakeet, Cohere, Orukeet) | Yes                | -                                        | No account, no limit                   |
| Local AI models / local agent                            | Yes                | -                                        | No account, no limit                   |
| Local speaker diarization                                | Yes                | -                                        | No cloud required                      |
| Local notes, folders, semantic search                    | Yes                | -                                        | Local SQLite                           |
| Custom dictionary, 100+ languages                        | Yes                | -                                        | Free                                   |
| BYOK cloud transcription                                 | Yes                | -                                        | Your provider bills you                |
| OpenWhispr Cloud transcription                           | 2,000 words/week   | Pro: unlimited                           | Hosted cost                            |
| Meeting recordings                                       | 5 hours/month      | Pro: 20 hours/month; Business: unlimited | Hosted cost                            |
| Cross-device sync                                        | -                  | Pro                                      | Hosted cost                            |
| Public note sharing                                      | -                  | Pro                                      | Hosted cost                            |
| Personal API / MCP                                       | -                  | Pro                                      | Hosted cost                            |
| Hosted CLI login                                         | -                  | Pro                                      | Local CLI commands are free            |
| Team creation inside a workspace                         | -                  | Business                                 | Teamspaces themselves are free         |
| Connectors (GitHub/Gmail/Slack/Linear/email)             | **Should be free** | Currently Pro                            | Client-side OAuth, no hosted inference |

### Where the gates are

- Word limit: `src/hooks/useUsage.ts:239-246`; `src/components/ControlPanel.tsx:418-440` opens the upgrade prompt.
- Upload size limit: `src/components/notes/UploadAudioView.tsx:108-109`, `417`, `634`.
- Team creation: `src/lib/capabilities.ts:72-84` (`create_teams === false` -> paywall); copy in `src/components/workspace/TeamPaywallDialog.tsx`.
- Connectors: `src/utils/connectorEligibility.ts` (deleted by PR #3258), `src/hooks/useConnectorAccess.ts:27`, `src/components/chat/chatTurnEngine.ts:733`.
- API/MCP/CLI: `src/components/integrations/integrationsSections.ts:52-58`.

## 2. What is free on Windows

Windows uses the same desktop code paths. The Windows build (`electron-builder.json:181-197`) bundles the local binaries (`resources/bin/windows-*`, `whisper-*`, `llama-server-*`, `sherpa-onnx-*`, `qdrant-*`) and the app defaults to local-capable modes.

Free on Windows, no account:

- Local Whisper and Parakeet transcription; CUDA/Vulkan GPU acceleration (`LOCAL_WHISPER_SETUP.md`).
- Local LLM cleanup/agent and local semantic search.
- Local speaker diarization (ONNX Runtime is bundled for Windows).
- Local notes, folders, custom dictionary, audio import.
- BYOK cloud providers.
- System audio loopback for meeting capture (`src/utils/systemAudioAccess.ts:22-31` returns `granted: true, mode: "loopback"` on `win32`).

The desktop pricing page is the same on every OS, so Windows has no platform-specific paid feature. The Windows gaps found are quality/bug issues already tracked upstream, not monetization.

### Known Windows issues in the upstream tracker (not new)

- #2059 elevated windows: dictation silently fails.
- #829 `windows-fast-paste.exe` fails for long transcripts (>~200 chars).
- #3099 microphone opens at 44100 Hz and captures no audio.
- #2675 Pause key fails global shortcut registration.
- #2265 GPU acceleration disabled after lid close/power saving.
- #3152 floating icon hit target is too large (PR #3234 is open).
- #3106 `windows-key-listener.exe` false-positive trojan detection.
- #1546 system audio output device selection.
- #2242 key listener leftover messages/unused IPC.
- #1641 remote desktop pastes as "v".
- #2468 GPU banner stays after CUDA install.
- #1418 whisper-server crashes on Ivy Bridge.

## 3. Artificial gate: connectors

Connectors are the one clear violation of "free except cloud services."

- `src/utils/connectorEligibility.ts:4` said "Connectors are paid-only."
- `src/hooks/useConnectorAccess.ts:27` required `isSignedIn && hasConnectorPlan(...)`.
- `src/components/chat/chatTurnEngine.ts:733-735` only offered connector tools to paid users.
- The connector code is local:
  - GitHub device flow + `https://github.com/login/oauth/access_token` (`githubDeviceFlow.js`, `githubApi.js:270-271`).
  - Gmail PKCE loopback + `https://gmail.googleapis.com/gmail/v1/users/me/messages/send` (`gmailAuth.js`, `gmailApi.js:9`).
  - Slack PKCE loopback + `https://slack.com/api/` (`slackAuth.js`, `slackApi.js:9`).
  - Linear PKCE loopback + `https://api.linear.app/graphql` (`linearAuth.js`, `linearApi.js:17`).
  - Email compose via the user's own mail handler (`emailCompose.js`).
- Slack/Linear use a stateless `openwhispr.com` redirect relay because those providers require HTTPS redirect URIs; the relay does not host connector execution or pay for provider usage.

**Issue:** #3246. **PR:** #3258 (makes connectors free for signed-in users, keeps org policy, keeps cloud/API/MCP/team features paid).

### Related entitlement bug

`hasConnectorPlan()` trusted `localStorage.getItem("isSubscribed") === "true"` whenever usage had not loaded (`src/utils/connectorEligibility.ts:9-11`, `src/lib/reactiveLocalFlag.ts:14`). A user could set that key in DevTools, and a canceled subscriber kept access offline. Removing the gate (PR #3258) removes the bypass. **Issue:** #3247.

## 4. Documentation and pricing-copy defects

- **README does not disclose the Cloud allowance or paid plans** (#3248, PR #3255). It says "The open-source and free alternative" and "MIT" but has no free-vs-paid section.
- **"Mobile app" is listed as a Pro feature** (#3249) in `src/locales/en/translation.json:2535`, while `openwhispr-mobile/README.md:66` says BYOK needs no Pro subscription.
- **`LOCAL_WHISPER_SETUP.md:95-98` says Cloud sends audio "to OpenAI" and costs "API usage"** (#3250, PR #3256). It should distinguish OpenWhispr Cloud (free allowance, then subscription) from BYOK (provider pricing).

These are the issues a Windows user hits first when they try to work out whether the app can be used for free.

## 5. Missing usage meter for meeting hours

The Free plan advertises 5 hours/month and Pro 20 hours (`src/locales/en/translation.json:2510`, `2530`), but `UsageData` and the Settings meter only track words (`src/lib/usageStore.ts:36-45`, `src/hooks/useUsage.ts:239-246`, `src/components/SettingsPage.tsx:2517-2555`). A user cannot see the meeting allowance being consumed. **Issue:** #3251.

## 6. Android sync status

**Verdict: partially synced in code, not at parity, and not validated in CI.**

| Capability                        | iOS                      | Android              | Evidence                                                                           |
| --------------------------------- | ------------------------ | -------------------- | ---------------------------------------------------------------------------------- |
| OpenWhispr Cloud                  | Yes                      | Yes                  | `inferenceModes.ts`                                                                |
| On-device Whisper (whisper.rn)    | Yes                      | Yes                  | `LocalWhisperService.ts`                                                           |
| On-device Parakeet                | Yes                      | No                   | `localEngine.ts:8`; `localModelCatalog.ts:130-138` hides Parakeet when unavailable |
| BYOK provider setup               | Yes                      | No                   | `inferenceModes.ts:53`; `AIModelsScreen.tsx:91`; README:72                         |
| On-device text cleanup/notes/chat | Yes (Apple Intelligence) | No                   | `localReasoning.ts`; `LocalReasoningService.ts`                                    |
| System-wide dictation keyboard    | Yes                      | No (floating bubble) | iOS keyboard extension; Android bubble PRs                                         |
| Google Calendar                   | Yes                      | No                   | `AccountScreen.tsx:34`                                                             |

Consequences:

- Android can transcribe on-device with Whisper, but cleanup, note formatting, note actions, note chat, and the voice assistant have no local engine. They use OpenWhispr Cloud.
- Android cannot use BYOK, so there is no free way to point those text workflows at a user-paid provider.
- A "fully free" Android setup is therefore limited to local transcription plus Cloud calls within the free allowance.

**Issues:** #3252 (Android local text gap), #3253 (Android bundle not validated). **PRs:** #3257 (documented parity table), #3259 (Android JS bundle in Mobile CI).

The existing #2741 covers the broader Android release request and already mentions the CI/BYOK gaps; #3252/#3253 split out the specific on-device text and bundle-validation work.

## 7. Issues filed

| #    | Title                                                                                            | Kind                   | PR           |
| ---- | ------------------------------------------------------------------------------------------------ | ---------------------- | ------------ |
| 3246 | Connectors are behind a paid plan even though they run client-side with the user's own accounts  | Free-feature violation | #3258        |
| 3247 | Connector entitlement is enforced by a localStorage flag and can outlive a canceled subscription | Security/entitlement   | #3258        |
| 3248 | README does not state what is free vs. paid, or the Cloud free allowance                         | Docs                   | #3255        |
| 3249 | Pricing lists "Mobile app" as a Pro feature, but the mobile app and BYOK are free                | Copy                   | -            |
| 3250 | Local Whisper setup's privacy/cost table says Cloud sends audio to OpenAI and costs "API usage"  | Docs                   | #3256        |
| 3251 | Free plan advertises 5 meeting hours/month, but the app has no meeting-hours meter               | Product/usage          | -            |
| 3252 | Android Local Mode has no on-device text model, so cleanup/notes/chat fall back to Cloud         | Parity                 | #3257 (docs) |
| 3253 | Mobile CI validates an iOS bundle only; Android bundling is not checked                          | CI                     | #3259        |

## 8. Pull requests opened

| PR   | Title                                                                  | Scope                                    |
| ---- | ---------------------------------------------------------------------- | ---------------------------------------- |
| 3255 | docs: add a free vs paid feature matrix                                | `README.md` only                         |
| 3256 | docs: correct the local vs cloud privacy and cost table                | `LOCAL_WHISPER_SETUP.md` only            |
| 3257 | docs(mobile): document iOS and Android platform parity                 | `openwhispr-mobile/README.md` only       |
| 3258 | feat(connectors): make client-side connectors free for signed-in users | Connector access, chat, UI badges, tests |
| 3259 | ci(mobile): bundle Android alongside iOS                               | `mobile-ci.yml` only                     |

## 9. Recommendations

1. **Merge or adapt #3258.** It is the only change that directly makes a non-cloud feature free. If maintainers prefer to keep connectors paid, the minimum fix is to enforce the entitlement server-side and remove the localStorage fallback.
2. **Adopt the free-vs-paid matrix (#3255) and corrected local/Cloud table (#3256).** These prevent the recurring "is this app actually free?" confusion.
3. **Fix the "Mobile app" copy (#3249) in all locales.** It is a one-line semantic fix per locale, but it needs a translator pass.
4. **Add a meeting-hours meter (#3251)** or remove the numeric meeting quotas from the pricing copy.
5. **Track Android parity explicitly (#3252/#3253).** The Android work is active; a parity table and an Android JS bundle check are cheap and prevent regressions.
6. **Windows: keep local-first defaults.** No monetization change is needed; the remaining work is the existing bug list in section 2.

## 10. Verification log (2026-11-16)

- PR #3255 (README): quality passed.
- PR #3256 (LOCAL_WHISPER_SETUP): quality passed.
- PR #3257 (mobile parity docs): Mobile static initially failed Prettier; fixed and re-run, static and bundle jobs pass.
- PR #3259 (Android bundle): Mobile bundles passed, including the new Android export.
- PR #3258 (connectors): the quality job was still running at the time of this report. The changed test files pass `node --check`, and Prettier reports no issues.

## 11. Method and caveats

- Code evidence is from the fork at `2eb67aa55` and upstream `main` at `e6834b20d` (2026-11-16).
- The desktop app's API server is not in this repository, so server-side entitlement behavior could not be inspected. Claims about server enforcement are limited to what the client code shows.
- The mobile app's Android native build was not run (no Android SDK/emulator in this workspace). Android conclusions come from platform branches, `Platform.OS` gates, native module availability checks, and the documented release status.
- No automated test suite was run in this checkout because `node_modules` is absent. The PRs that touch tests rely on upstream CI for the final signal.
