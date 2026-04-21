# Localization and Internationalization Strategy

**Covers:** All language and regional adaptation for X-Ray Vision Diagnostics
**Current languages:** English (US, UK), Spanish (Mexican), French (Quebec)
**Planned languages:** German (Phase 6+), Japanese (Phase 6+)

---

## Philosophy

We do not localize everything at once. We localize the minimum viable surface to serve each market's legal and functional requirements, then expand based on user demand and NPS feedback. For safety-critical products, a poor translation is worse than no translation — we maintain a "certified translation" bar for any safety-related text.

**Safety text is never machine-translated.** All HV-STOP messages, safety warnings, and OSHA-required documentation are translated by a certified technical translator and reviewed by a native-speaking safety engineer or compliance professional before deployment.

---

## Language Coverage Matrix

| Surface | English US | English UK | Spanish (MX) | French (Quebec) |
|---|---|---|---|---|
| Headset UI (step cards, labels) | ✓ | ✓ (minor vocabulary delta) | ✓ | ✓ |
| Voice wake word | "Hey Aria" | "Hey Aria" | "Oye Aria" | "Allez Aria" |
| Voice command vocabulary | Full | Full | Full (MX automotive) | Full (Canadian French automotive) |
| HV-STOP alert text | ✓ | ✓ | ✓ | ✓ |
| Safety warnings (all) | ✓ | ✓ | ✓ | ✓ |
| Onboarding program materials | ✓ | ✓ | Planned Phase 5 | Planned Phase 5 |
| Manager dashboard | ✓ | ✓ | Phase 6 | Phase 5 (Quebec requirement) |
| OEM vocabulary (automotive terms) | Full | UK delta only | Toyota/Ford MX vocab | Toyota Canada/Ford Canada |
| Legal / compliance docs | ✓ | UK law adapted | ✓ (OSHA ES equivalent) | ✓ (Quebec labor code) |

---

## VoiceNLP Localization Architecture

VoiceNLP uses a language-specific model stack per locale. The architecture is:

```
Wake word detection → Language identification → Locale-specific ASR → Shared intent classifier → Locale-specific entity extractor
```

**Wake word detection:** Each locale has its own wake word model (tiny, runs on headset). "Hey Aria" and "Oye Aria" are acoustically distinct enough that the detector does not require locale pre-selection — it runs all wake word models in parallel and fires on first match.

**Language identification:** After wake word, a language identification model (< 5ms, runs on edge server) classifies the incoming audio as English/Spanish/French. This routes to the correct ASR model.

**ASR model:** Locale-specific fine-tuned Whisper model, optimized for:
- Bay-voice (louder, more deliberate than conversational speech)
- High ambient noise (85–95 dB compressor background)
- Automotive domain vocabulary (OBD-II codes, component names, OEM terminology)

**Intent classifier:** Shared across all locales (intents are language-agnostic: SHOW_LAYER, NEXT_STEP, HV_STOP_ACKNOWLEDGE, etc.)

**Entity extractor:** Locale-specific vocabulary for component names, OEM terms, DTC codes.

### OEM Vocabulary per Locale

| Term (component) | English US | English UK | Spanish (MX) | French (Quebec) |
|---|---|---|---|---|
| Hood | Hood | Bonnet | Capó | Capot |
| Trunk | Trunk | Boot | Cajuela / Maletero | Coffre |
| Tire | Tire | Tyre | Llanta | Pneu |
| Battery coolant pump | Battery coolant pump | Battery coolant pump | Bomba de refrigerante de batería | Pompe de refroidissement de batterie |
| High-voltage cable | HV cable / orange cable | HV cable / orange cable | Cable de alto voltaje / cable naranja | Câble haute tension / câble orange |
| Thermal management system | TMS | TMS | Sistema de gestión térmica / SGT | Système de gestion thermique / SGT |

Full vocabulary files are maintained per OEM × locale: `docs/localization/vocab/{oem}_{locale}.json`

---

## UI Localization

### String Externalization
All UI strings in the headset application and manager dashboard are externalized to locale-specific JSON files:
```
/localization/
  en-US.json
  en-GB.json
  es-MX.json
  fr-CA.json
```

Locale selection is determined at the device level (headset provisioned with a locale at install time by the field engineer). A tech can override their headset locale via manager dashboard.

### Character Set and Rendering
- English/French/Spanish: Latin character set — HoloLens 2 default font supports all required glyphs
- French special characters (é, è, ê, à, â, ç, œ): confirmed rendering correctly in Mixed Reality Toolkit (MRTK) text renderer; tested in Phase 5 French localization sprint
- Right-to-left languages: not currently in scope; would require MRTK text layout changes (Phase 6+ assessment)

### Text Expansion
French and Spanish strings are typically 20–40% longer than English equivalents. Step card layout must accommodate expansion without truncation. Design rule: all step card text boxes sized for 140% of English string length.

---

## Safety Text Translation Process

**Requirement:** Any text that appears on the HoloLens display during an HV-STOP event, proximity warning, or connection-loss warning must be translated by a certified technical translator and reviewed by a native-speaking safety professional.

**Process:**
1. English safety strings finalized and locked (no changes after this point without re-translation)
2. Send to certified technical translator (specialization: automotive or industrial safety)
3. Back-translation review: a second independent translator back-translates to English; compare with original
4. Native-speaking safety engineer reviews back-translation for meaning accuracy
5. Legal review: compliance manager confirms translated safety text meets applicable regulations in target market
6. Approved strings committed to locale JSON file

**Turnaround time:** Allow 4 weeks for safety text translation. Non-safety strings: standard 2-week sprint.

---

## Quebec French Requirements (Special Case)

Quebec's Charter of the French Language (Bill 101) and the updated Law 25 (privacy) create specific requirements for Quebec deployments:

1. **Headset UI must be in French** for any Quebec-based deployment — English is not acceptable as the primary language even if the tech prefers English
2. **Manager dashboard**: French-language version required for any Quebec dealership account
3. **Onboarding materials**: French versions required (cannot distribute English-only materials to Quebec dealers)
4. **Privacy consent**: Quebec Law 25 requires explicit French-language consent banners for personal data collection

Implementation: Quebec is provisioned as a distinct locale (`fr-CA-QC`) that differs from standard Canadian French only in the consent banner wording (Law 25 specific language).

---

## Language Expansion Roadmap

### Phase 5 (Current)
- ✓ English US, UK
- ✓ Spanish (Mexican — covers 19% of US automotive technician workforce)
- ✓ French Quebec (required for Canada market entry)

### Phase 6 Planning (Year 3–4, international expansion)
- **German:** Required for Germany (Volkswagen Group OEM dealer network); 12-week localization sprint; automotive vocabulary heavily compound-word — requires dedicated entity extractor fine-tuning
- **Japanese:** Required for Japan market (Toyota home market); right-to-left layout changes not required (Japanese is top-to-bottom but MRTK handles this); Kanji + Hiragana character rendering requires font switch
- **Korean:** Required for Hyundai/Kia OEM partnership (Phase 6 target); similar complexity to Japanese

### Localization Quality Bar by Language Maturity

| Maturity Level | Definition | Languages |
|---|---|---|
| Production | Full coverage, native speaker QA, safety text certified | English US, English UK, Spanish MX |
| Beta | Full coverage, safety text certified, some UI polish remaining | French Quebec |
| Planned | Vocabulary list started, no ASR model yet | German, Japanese |
| Roadmap | Not started | Korean, Portuguese (Brazil) |

---

## Localization Testing Protocol

Before any locale ships to production:

1. **Voice recognition rate test:** ≥ 97% on a 200-command test set recorded by 5 native speakers in a simulated bay environment (not a studio)
2. **Safety text back-translation review:** Pass (no meaning deviation)
3. **UI rendering test:** All screens rendered at 140% English string length — no truncation, no overlap
4. **Wake word false-positive test:** Wake word must not fire on common non-command phrases in that language (e.g., "Oye" alone in Spanish must not trigger "Oye Aria" — tested with 50 ambient audio clips)
5. **End-to-end session test:** One complete repair session in locale, all steps, including HV-STOP trigger and acknowledgment, performed by a native speaker
