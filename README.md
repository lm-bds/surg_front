# surg_front

SvelteKit UI for a surgical scheduling dashboard. Clinicians add upcoming surgeries
(surgeon, procedure, diagnosis, planned start) and the app submits them to a backend
for duration prediction and scheduling.

This frontend is the presentation layer for the **Surg_sim** engine (Rust), which
simulates operating-room throughput using a feature-aware duration estimator.

## Status

- Functional UI with additive surgery cards, form validation, and a results view.
- Wired to POST to `/api/predict-duration` (configurable via `VITE_API_BASE`).
- The backend it talks to lives in the private `Surg_sim` repository.

## Develop

```bash
npm install
npm run dev
```

Set `VITE_API_BASE` to point at the scheduling service before running predictions.
