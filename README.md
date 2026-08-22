# surg_front

A typed SvelteKit client for the [Surg_sim](https://github.com/lm-bds/Surg_sim) illustrative duration-prediction API.

## Run

Start the Rust API on port 3001, then:

```bash
npm ci
npm run dev
```

Set `VITE_API_BASE` when the API is hosted elsewhere:

```bash
VITE_API_BASE=https://example.test npm run dev
```

## Verify

```bash
npm run check
npm run lint
npm run build
npm audit
```

The interface validates each case, handles API and schema errors, and displays the returned duration predictions. The bundled model coefficients are demonstrations and are not clinically validated.

## Licence

MIT.
