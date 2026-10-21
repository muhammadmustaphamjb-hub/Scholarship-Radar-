# Firestore Data Schema v0.1

Conventions
- Unknown values are stored as `null` and the matching `*Status` field is `"unknown"`. Never fabricate.
- Dates are Firestore Timestamps (UTC). Deadlines also store the original text and timezone if the source gave one.
- Money is stored as `{ amount: number|null, currency: string|null }` (ISO 4217).

---

## users/{uid}  (private, owner only)
- fullName: string
- email: string
- preferredLanguage: "en"|"ha"|"ar"|"fr"|"zh"|"es"|"pt"
- nationality: string|null (ISO 3166-1 alpha-2)
- countryOfResidence: string|null
- academicLevel: "undergraduate"|"bachelor"|"master"|"phd"|"postdoc"|null
- discipline: string|null
- researchInterests: string[]
- cgpa: { value: number, scale: number }|null
- englishTest: { type: string, score: number }|null  // optional
- eliInstructionCertificate: boolean|null  // English-medium instruction
- fundingPreferences: string[]
- createdAt, updatedAt: Timestamp

### users/{uid}/saved/{opportunityId}
- savedAt: Timestamp, note: string|null

### users/{uid}/applications/{opportunityId}
- status: "planning"|"in_progress"|"submitted"|"result_received"
- checklist: [{ item: string, done: boolean }]
- updatedAt: Timestamp

### users/{uid}/notifications/{id}
- type, opportunityId, createdAt, read: boolean

---

## opportunities/{id}  (public read, backend/admin write)
- kind: "scholarship"|"research_grant"|"fellowship"|"assistantship"|"postdoc"
- title: string
- fundingOrganization: string
- hostInstitution: string|null
- country: string|null
- degreeLevels: string[]
- disciplines: string[]
- eligibleNationalities: { mode: "all"|"include"|"exclude"|"unknown", list: string[] }
- eligibilityCriteria: [{ id, text, type }]  // each criterion is evaluated Met/Not met/Unknown per user
- academicRequirement: string|null
- languageRequirements: { text: string|null, acceptsEMI: "yes"|"no"|"unknown", acceptedTests: string[] }
- opensAt: Timestamp|null
- deadline: Timestamp|null
- deadlineText: string|null
- coverage: {
    tuition: "yes"|"no"|"partial"|"unknown",
    stipend: { status: "yes"|"no"|"unknown", amount: Money|null, period: string|null },
    accommodation: "yes"|"no"|"partial"|"unknown",
    travel: "yes"|"no"|"partial"|"unknown",
    healthInsurance: "yes"|"no"|"unknown"
  }
- fundingCategory: "fully_funded"|"partially_funded"|"tuition_waiver"|"unknown"
- officialAnnouncementUrl: string|null
- officialApplicationUrl: string|null
- originalPublishedAt: Timestamp|null
- lastVerifiedAt: Timestamp|null
- lastUpdatedAt: Timestamp
- openStatus: "open"|"upcoming"|"closed"|"suspended"|"renewed"|"unknown"
- verificationStatus: "officially_verified"|"official_source_review_required"|"unverified_lead"|"expired_or_closed"|"possibly_fraudulent_review_required"
- contentHash: string  // duplicate detection
- missingFields: string[]

### Research-grant extras (when kind != scholarship)
- grant: {
    eligibleApplicants: string|null,
    eligibleCountries: { mode, list },
    amount: Money|null, durationMonths: number|null,
    affiliationRequired: "yes"|"no"|"unknown",
    supervisorOrEndorsementRequired: "yes"|"no"|"unknown",
    eligibleExpenses: string[]
  }

### opportunities/{id}/evidence/{evidenceId}  (backend/admin write)
- sourceUrl: string
- sourceOrganization: string
- isOfficialDomain: boolean
- checkedAt: Timestamp
- supportingText: string  // short excerpt or summary of what supports which field
- fieldsSupported: string[]
- accessible: boolean  // false if official source could not be reached

### opportunities/{id}/verificationHistory/{eventId}  (append-only, backend/admin)
- previousStatus, newStatus: string
- changedAt: Timestamp
- changedBy: { type: "automated"|"human", id: string }
- missingInformation: string[]
- warningSigns: string[]  // required when newStatus is possibly_fraudulent_review_required
- evidenceIds: string[]

---

## Backend-only collections (no client access)
- sources/{id}: adapter name, base URL, terms/permission note, rate limit, enabled
- sourceRuns/{id}: sourceId, startedAt, finishedAt, itemsFound, errors[]
- reviewQueue/{id}: opportunityId, reason, createdAt, assignedTo
- admins/{uid}: role
