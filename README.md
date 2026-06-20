# ESG Answer

Free ESG/carbon self-check tool for Indian SME suppliers who receive an ESG
questionnaire from a large customer (driven by SEBI BRSR value-chain rules and
EU CSRD/VSME). Helps a supplier turn their electricity and fuel bills into a
Scope 1 + Scope 2 carbon summary — using India's CEA grid emission factor —
formatted to send straight back to the customer.

**Site:** esganswer.com
**Positioning:** the *answer brand* (content + workflow + trust), not "another carbon calculator".

## Structure
- `index.html` — the shippable site: the working Supplier ESG Self-Check tool + guide + FAQ. Fully self-contained (no build step, no dependencies beyond Google Fonts).
- `docs/hub-article.md` — the hub article draft (the panic-search landing page).
- `docs/calculator-spec.md` — the tool build spec: inputs, India emission factors, formulas, output. **Not deployed.**

## Before launch
1. **Verify `EF.elec`** in `index.html` against the latest CEA CO₂ Baseline Database (V21) — this is the India-correct grid factor and the core differentiator.
2. Optionally wire email capture + PDF export (currently uses browser print-to-PDF).
3. Deploy static (GitHub Pages / Cloudflare Pages / Firebase) and point esganswer.com DNS.

## Local preview
```
python -m http.server 8731
# open http://localhost:8731
```

Independent, sponsor-free. No fabricated numbers; no user data stored (the tool runs entirely in the browser).
