# Global Scholarship & Research Funding AI Platform — Architecture v0.1

Status legend used in this document:
- DESIGNED = decided on paper, no code yet
- NEEDS CREDENTIALS = cannot work until the owner creates an account or key
- NEEDS PAID SERVICE = free tier may be insufficient

Nothing in this file claims that an app has been compiled or tested on a device.

---

## 1. Key constraint: phone only, no computer

The owner has an Android phone and no computer. This decides the workflow:

| Task | Can be done from a phone browser? | How |
|---|---|---|
| Write and edit code | Yes | GitHub Codespaces or github.dev in the mobile browser (awkward but possible; desktop-site mode recommended) |
| Compile the APK | Yes | GitHub Actions builds the APK in the cloud; the APK is downloaded to the phone |
| Test the app | Yes | Install the debug APK on the owner's own phone |
| Create backend (Firebase project) | Yes | Firebase console works in a mobile browser |
| Publish to Google Play | Partly | Needs a Google Play Developer account (one-time USD 25 fee, identity verification). Console works on mobile but is tedious |
| Sign the release build | Yes | Keystore generated in GitHub Actions, stored as encrypted secret. Losing it means losing the ability to update the app |

Honest warning: building a Compose Android app entirely from a phone is possible but slow. A cheap used laptop would remove most friction. This is not required, only recommended.

## 2. Technology decision

### Chosen: native Android, Kotlin + Jetpack Compose + Material 3

Why:
- Best fit for a Google Play release with good performance on small, low-end devices.
- Compose supports right-to-left layouts (Arabic) and per-app locale switching natively.
- First-class Firebase SDKs.

Rejected alternatives and trade-offs:
- Flutter / React Native: one codebase would help if iOS is wanted later, but adds a second toolchain to learn and debug from a phone. Reconsider if iOS becomes a requirement.
- Web app / PWA only: far easier to build from a phone, but the owner asked for a Google Play app. A PWA can be a stepping stone (see roadmap, Phase 0).

### App architecture
- Pattern: MVVM with unidirectional data flow, repository layer.
- Kotlin Coroutines and Flow.
- Dependency injection: Hilt (justified: several repositories, testability).
- Navigation: Navigation Compose.
- Local cache: Room (offline state for saved opportunities).
- Localization: Android resource files (`values-ha`, `values-ar`, `values-fr`, `values-zh`, `values-es`, `values-pt`) plus `AppCompatDelegate.setApplicationLocales` for in-app language selection and persistence.

### Backend: Firebase (Spark free tier to start, Blaze later)
- Authentication: email/password, email verification, password reset, Google Sign-In.
- Cloud Firestore: opportunity records, user profiles, saved items, tracking.
- Cloud Functions: discovery jobs, AI extraction, verification, notifications. NEEDS PAID SERVICE: Functions require the Blaze plan (pay-as-you-go, a billing card is required even though small usage is usually near zero).
- Firebase Cloud Messaging: deadline notifications.
- Firebase App Check: reduces abuse of the backend from non-genuine apps.

Why Firebase: no server to maintain, managed from a browser, security enforced by rules. Trade-off: Firestore is weak at complex text search. Plan: filters in Firestore first; add a search service (Algolia, Typesense, or Firestore vector search) when needed.

## 3. Security rules of the design (non-negotiable)

1. No private API key or service-account credential ever ships inside the APK. AI model keys live only in Cloud Functions secrets.
2. The app talks to AI only through a Cloud Function that checks the user is authenticated and rate-limits them.
3. Passwords are handled by Firebase Auth; the project never stores them.
4. Firestore rules enforce that a user can read and write only their own private data (see `firestore.rules`).
5. Opportunity and verification records are writable only by the backend or admins, never by ordinary users.
6. Account deletion removes the auth account and all user-owned documents (Cloud Function triggered on user deletion).
7. External content (web pages, PDFs, social posts) is untrusted data. The extraction function treats it as text to parse, never as instructions, and has no tool access beyond writing a draft record.

## 4. Discovery engine (DESIGNED)

Pipeline: Source adapter -> Fetch -> Extract (AI + schema validation) -> Deduplicate -> Verify -> Publish.

- Each source is an adapter implementing one interface (`fetch()`, `parse()`), so new sources do not require rewrites.
- Allowed sources first: official university and government scholarship pages, funding agency pages, public RSS feeds, authorized APIs.
- Social platforms (LinkedIn, Facebook, Instagram, X, Indeed): most restrict access or scraping in their terms and APIs, and some APIs are paid or approval-gated. They are NOT part of the first release. They can be added later only through officially permitted access. Posts from social media are always stored as UNVERIFIED LEAD.
- Scheduling: Cloud Scheduler triggers per-source jobs. Incremental updates by content hash. Retries with backoff. Per-source rate limits. Failures logged to a `sourceRuns` collection and surfaced to admins.
- Duplicate detection: normalized title + organization + deadline + official URL hash.

## 5. AI extraction and assistant (DESIGNED)

- Extraction outputs JSON validated against a strict schema. Unknown fields remain `null` with `unknown` status, never guessed.
- Dates, currencies and URLs are validated in code, not by the model.
- Assistant uses retrieval over the app's own database (filters first, then semantic ranking), answers cite the stored record and its official link, and states plainly when the database has no match. It never claims to have searched the whole internet.
- Eligibility is shown per criterion: Met / Not met / Unknown. The app never says "you qualify" from partial matches.
- NEEDS CREDENTIALS and NEEDS PAID SERVICE: an AI model API key (Anthropic or another provider) with usage-based billing.

## 6. Verification engine (DESIGNED)

Statuses: `officially_verified`, `official_source_review_required`, `unverified_lead`, `expired_or_closed`, `possibly_fraudulent_review_required`.

Rules:
- `officially_verified` requires a stored evidence record from the official organization's own domain, checked on a recorded date.
- `possibly_fraudulent_review_required` requires listed warning signs (for example fee demands, look-alike domains, mismatched organization) and always goes to human review. No automatic "fraud" label is shown to users as fact.
- Every status change appends a history record (previous status, new status, who or what, date, evidence).
- AI confidence alone never promotes a record to verified.

## 7. Multilingual plan

Languages: English, Hausa, Arabic, French, Chinese, Spanish, Portuguese.
- UI strings in resource files. English is the source. Other languages start as machine-assisted drafts and are marked unreviewed until a fluent speaker checks them. Coverage will not be claimed as complete until every string has been checked.
- Arabic: full RTL layout testing required.
- Scholarship descriptions are stored in the original language; any translation is labelled "automatic translation, may be inaccurate" with a link to the official source text.

## 8. Roadmap

- Phase 0 (week 1): Create GitHub repo, Firebase project, GitHub Actions workflow that builds a debug APK. Goal: a "Hello" app installs on the owner's phone.
- Phase 1: Auth (sign up, login, logout, verification, reset, Google Sign-In), profile, language selector.
- Phase 2: Opportunity database, manual admin entry, list, search, filters, detail screen, saved items.
- Phase 3: Verification workflow and evidence records, admin screens.
- Phase 4: Discovery adapters (official sources first), AI extraction.
- Phase 5: AI assistant with retrieval, citations.
- Phase 6: Notifications, application tracker, privacy policy, Play Store listing, closed testing (Google requires testing with real testers for new personal developer accounts before production release).

## 9. What the owner must provide or decide

1. A GitHub account (free).
2. A Google account for the Firebase project (free to start; billing card needed for Cloud Functions).
3. A Google Play Developer account (USD 25) and identity verification, when ready to publish.
4. An AI API key and billing, when Phase 4 starts.
5. A privacy policy page (required by Google Play because the app collects personal data). It can be drafted during Phase 6.
6. An app name and a decision on whether to keep it free, ad-supported, or paid.

## 10. Open questions

- App name?
- Should the first release be English plus Hausa only, then add the others, to keep translation quality high?
- Will the owner have access to any computer, even occasionally, for signing and Play Console steps?
