# http2checker — Agent Notes

## Single-file app
- `http2checker.py` is the sole source — no package manager, no dependencies beyond stdlib.
- No tests, no linting, no typecheck, no CI. No `.gitignore`.
- No Makefile, no build step.

## Run
```bash
python3 http2checker.py <domain>
python3 http2checker.py -f domains.txt -k -o results.csv
```

Arguments: `-k` (skip SSL verify), `-f` (batch from file, `#` for comments), `-v` (show exception tracebacks), `-o` (CSV export).

## Architecture
- Entrypoint: `main()`. `analyze_domain()` performs all checks per domain.
- Sequence: DNS → HTTP HEAD → H2 detection (separate connect with ALPN `['h2', 'http/1.1']`) → HTTPS + SSL cert (ALPN `['http/1.1']` only) → page title (follows redirects).
- H2 detection uses a separate connection to avoid `BadStatusLine` when a server negotiates h2 but receiving an HTTP/1.1 request.
- SSL Status categories: `VALID`, `EXPIRED`, `SELF-SIGNED`, `ERR_CERT`, `ERR_HOSTNAME`, `ERR_TLS_VERS`, `ERR_SSL`, `BYPASS (-k)`, `TIMEOUT`, `REFUSED`, `NO_HTTPS`.
- Terminal output truncates long values for alignment; CSV export writes full values and calls `flush()` on every line to prevent data loss on crash.

## License
GPL v3 — see `LICENSE`.
