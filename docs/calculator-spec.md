# Supplier ESG Self-Check — Calculator Build Spec

The free tool that anchors the whole venture. One page, no login, ~5 minutes.
Build it exactly like a wibest calculator: a static HTML page + vanilla JS, all
factors baked into a JS object. The differentiator is **India-correct factors +
a BRSR/VSME-shaped output**, NOT the math (the math is simple).

---

## 1. Design principles
- **Plain language, zero jargon** on screen. Say "diesel for your generator," not "Scope 1 stationary combustion."
- **Accept what the user actually has:** let them enter electricity as **kWh OR ₹ on the bill** (convert ₹ → kWh with an editable tariff). Same for fuel: litres OR ₹.
- **Never block on a missing field.** Empty = 0, with a note "you can add this later."
- **Output is the product.** The number is table stakes; the *send-this-back-to-your-customer summary* is the reason they came.

---

## 2. Inputs

### Section E — Environment
| Field | Unit | Notes |
|---|---|---|
| Grid electricity | kWh/year *or* ₹/year + ₹/kWh tariff | Default tariff ₹8/kWh (editable) |
| Diesel (generator + vehicles) | litres/year *or* ₹ + ₹/litre | |
| Petrol | litres/year | |
| LPG | kg/year *or* no. of 19kg cylinders | 1 commercial cylinder = 19 kg |
| CNG / PNG (piped natural gas) | kg/year or m³/year | |
| Water | kL/year | Optional — reported, not carbon-scored |
| Waste to landfill | tonnes/year | Optional |

### Section S — Social (yes/no + number)
- Total employees; female employees (→ % women).
- Pay ≥ statutory minimum wage? (Y/N)
- Written safety / no-child-labour / anti-harassment policy? (Y/N each)

### Section G — Governance
- Company name, sector, point-of-contact email.
- Code of conduct in place? (Y/N)

---

## 3. Emission factors (bake these in; cite + date them)

> **IMPORTANT — verify before launch.** Pull the latest **CEA CO₂ Baseline Database (Version 21.0, Nov 2025, FY2024-25)** for the all-India grid factor and update the constant. The number below is the standard order of magnitude; confirm the exact published value and footnote it (BRSR Core requires the methodology footnote).

```js
// All factors in kg CO2e per unit. Sources noted per line.
const EF = {
  electricity_kwh: 0.71,   // kg CO2/kWh — CEA all-India grid EF (~0.71 tCO2/MWh). VERIFY vs CEA V21.
  diesel_litre:    2.68,   // kg CO2/litre — IPCC/DEFRA standard
  petrol_litre:    2.31,   // kg CO2/litre
  lpg_kg:          2.98,   // kg CO2/kg
  cng_kg:          2.69,   // kg CO2/kg  (natural gas; if entered as m3, ~0.78 kg/m3 density)
  // Water & waste: reported as activity data, not converted to CO2 in v1 (keep simple)
};
const NG_KG_PER_M3 = 0.78; // to convert PNG m3 -> kg if needed
```

**Why this matters / the moat:** the popular free calculators (SME Climate Hub, Carbon Trust, carbonfootprint.com) bake in **UK/US grid factors** (~0.2 kg/kWh). India's grid is ~3× more carbon-intensive (~0.71). A supplier using a Western tool **understates emissions by ~70%** and hands their customer a number that won't survive assurance. Being India-correct is the wedge.

---

## 4. The math (this is all of it)

```js
// Scope 2 — purchased electricity
const scope2 = elec_kwh * EF.electricity_kwh;

// Scope 1 — fuel burned directly on site / in own vehicles
const scope1 =
    diesel_litre * EF.diesel_litre +
    petrol_litre * EF.petrol_litre +
    lpg_kg       * EF.lpg_kg +
    cng_kg       * EF.cng_kg;

const totalKg = scope1 + scope2;
const totalTonnes = totalKg / 1000;          // present in tCO2e (rounded to 1 dp)
const intensity_per_employee = totalTonnes / employees; // a simple, defensible KPI
```

Round to 1 decimal. Show Scope 1 and Scope 2 **separately** — that split is exactly what BRSR/CSRD questionnaires ask for, and showing you know the difference signals competence.

---

## 5. Output — the "Supplier ESG Summary" (the actual product)

Render an on-screen card + a **download as PDF** (and "copy as table"). Structure it to map 1:1 onto a typical supplier questionnaire so the user can literally paste/attach it:

```
SUPPLIER ESG SUMMARY — [Company], FY2024-25
Prepared with esganswer.com · Methodology: GHG Protocol; grid factor CEA V21

ENVIRONMENT
  Scope 1 (direct fuel):        X.X tCO2e
  Scope 2 (electricity, CEA):   X.X tCO2e
  Total operational footprint:  X.X tCO2e
  Emissions intensity:          X.X tCO2e / employee
  Water:  X kL/yr   Waste to landfill: X t/yr
  Methodology note: Scope 2 uses CEA all-India grid EF (vXX). Scope 1
  per IPCC/DEFRA factors. [auto-filled]

SOCIAL
  Employees: N (W% women) · Minimum wage: Yes
  Policies: Safety [Y] · No child labour [Y] · Anti-harassment [Planned Q_]

GOVERNANCE
  Code of conduct: Yes · Contact: name@company

GAPS & PLAN (honest, dated)
  - Scope 3 not yet measured — planned FY__
  - Anti-harassment policy — drafting, target __
```

The "Planned" / "Gaps & Plan" lines are auto-generated from any **No** answers — turning weaknesses into the credible "not yet, here's the plan" framing procurement teams prefer.

---

## 6. Monetisation hooks (build the rails, don't sell hard in v1)
- **Email capture** to download the branded PDF (the lead).
- **Upsell slot:** "Want this on your letterhead, reviewed by an expert? ₹999." (manual fulfilment at first).
- **Affiliate slot:** when a user's footprint is large/complex, refer to an ESG consultant or the enterprise platforms that *want* clean supplier data.
- **Later SaaS:** save the evidence folder, auto-fill next year / next customer's questionnaire (the recurring-revenue version).

---

## 7. Build checklist (the weekend)
- [ ] Static page `/tools/supplier-esg-self-check/` (clone a wibest calculator shell).
- [ ] Inputs + the EF object above; confirm CEA V21 value.
- [ ] On-screen summary card + PDF export (use a tiny client-side lib or print stylesheet).
- [ ] Email-capture gate on PDF download.
- [ ] Publish the hub article; link the CTA to this tool (and back).
- [ ] JSON-LD: SoftwareApplication/WebApplication + FAQ — same discipline as wibest (run a validator before publishing).
- [ ] Plausible/GA: track tool-completions and PDF downloads as the validation metric.

**Validation gate (6–8 weeks):** are people finding the hub article on those keywords, and completing the tool? If yes → build article #2–#5 and the save/reuse feature. If no → you've spent a weekend, walk away clean.
