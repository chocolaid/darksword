Reference: https://cloud.google.com/blog/topics/threat-intelligence/darksword-ios-exploit-chain

This repository is a static, multi-stage JavaScript chain. The top-level pages only bootstrap the next stage, and most of the behavior is in large stage scripts.

## High-level flow

1. `index.html` stores a session value and creates a hidden iframe to `frame.html`.
2. `frame.html` injects `rce_loader.js`.
3. `rce_loader.js`:
   - Detects iOS version from `navigator.userAgent`.
   - Loads version-specific worker/module code (`rce_worker*.js`, `rce_module*.js`).
   - Starts a worker and coordinates stage messages (`prepare_dlopen_workers`, `trigger_dlopen*`, `sign_pointers`, etc.).
4. Worker stages eventually evaluate:
   - `sbx0_main_18.4.js` -> `sbx1_main.js` -> `pe_main.js`
5. Redirect behavior is triggered at the end of the worker sequence.

## File roles

- `index.html`: browser entry page.
- `frame.html`: iframe loader page.
- `rce_loader.js`: main orchestrator and version branch logic.
- `rce_module.js`: non-18.6 setup/device-offset logic.
- `rce_module_18.6.js`: 18.6-specific lightweight module.
- `rce_worker.js`, `rce_worker_18.6.js`: worker-side stage pipeline.
- `sbx0_main_18.4.js`, `sbx1_main.js`, `pe_main.js`: later stage payload chain.

## Version/device specificity

The code is heavily data-driven with large per-version and per-device offset tables in:

- `rce_module.js`
- `sbx0_main_18.4.js`
- `sbx1_main.js`

The loader uses iOS version checks to choose which worker/module pair to run.
