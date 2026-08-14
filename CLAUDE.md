# Working with this repo

One bash script, `download-ldc-corpora`, that downloads corpora from the LDC catalog.

## Running it

```
./download-ldc-corpora LDC96L14 [MORE_IDS ...]
```

Configuration comes from the environment, else `ldc.env` beside the script, else
`~/.ldc.env`:

| Variable | Meaning |
|---|---|
| `LDC_EMAIL` | LDC account email |
| `LDC_PASSWORD` | LDC account password |
| `LDC_DOWNLOAD_DIR` | Where files are written (default: current directory) |

If `ldc.env` does not exist, tell the user to `cp ldc.env.example ldc.env`, fill it in, and
`chmod 600` it. Do not create it for them with guessed values, and do not ask them to paste
credentials into the chat.

Exit codes: `0` ok, `1` config, `2` auth, `3` unknown corpus, `4` disk, `5` download.

## Rules

- **Never try to enter credentials interactively.** `read` gets EOF under this harness — the
  script would receive an empty password. That is exactly the bug this design fixes; the
  script now exits with code 1 instead. Fix the config, don't work around it.
- **Never print `ldc.env`, `cookies.txt`, or `$LDC_PASSWORD`,** and never commit them. All
  three are gitignored. `cookies.txt` is a live authenticated session.
- **Only download the corpus IDs the user named.** An LDC account can hold hundreds of
  corpora totalling many TB — far more than a typical filesystem has free. Never expand a
  request into "download everything".
- **Check size before large pulls.** After any run, `corpora.tsv` lists every corpus the
  account can download; column 1 is the ID, column 6 has the filename, size, and MD5.
  `grep '^LDC2023S02' corpora.tsv` shows the parts and their sizes.

## Behaviour worth knowing

- **A corpus is often several files.** The catalog lists one row per part, so a single ID can
  map to 20+ rows and hundreds of GB. All parts are downloaded. The same file can also appear
  twice under different URLs; those are deduplicated by (filename, MD5).
- **Re-running is safe and cheap.** Files already present with a matching MD5 are skipped
  without contacting the server. Interrupted transfers resume.
- **Most files have no extension in the catalog** — it is taken from the server's
  `Content-Disposition` header after download. Do not assume `.tgz`.
- **A few corpora have no published MD5** (the catalog says `calculating`); those are verified
  by exact byte count instead.
- **`.part` means incomplete.** A file is renamed to its final name only after verification.
- The session in `cookies.txt` is reused across runs, so most runs never hit the login form.
