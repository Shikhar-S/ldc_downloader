---
name: ldc-download
description: Download corpora from the LDC catalog by corpus ID (e.g. LDC96L14) using the credentials in ldc.env. Use when the user asks to fetch, download, or get an LDC corpus, or asks what LDC data their account can access.
---

# Download LDC corpora

Run `download-ldc-corpora` from the repo root with the corpus IDs the user asked for.

## Steps

1. **Check config.** If neither `ldc.env` (beside the script) nor `~/.ldc.env` exists, and
   `LDC_EMAIL`/`LDC_PASSWORD` are not already in the environment, stop and tell the user:

   ```
   cp ldc.env.example ldc.env && $EDITOR ldc.env && chmod 600 ldc.env
   ```

   Never ask them to paste credentials into the chat, and never invent values.

2. **Run it**, passing every requested ID in one invocation:

   ```
   ./download-ldc-corpora LDC96L14
   ```

   Large corpora take a while — run in the background and report when it finishes. The script
   logs each part as it goes and verifies checksums itself.

3. **Report** where files landed, their sizes, and any part that failed. On a non-zero exit,
   map the code: `1` config, `2` bad credentials, `3` corpus not on the account, `4` not
   enough disk, `5` a transfer or checksum failed (re-running resumes).

## Listing what is available

Any run refreshes `corpora.tsv` in the download directory: column 1 is the corpus ID, column
2 the title, column 6 the filename, size, and MD5. Grep it rather than downloading to explore:

```
grep -i celex "$LDC_DOWNLOAD_DIR/corpora.tsv" | cut -f1,2
```

## Constraints

- Only download the IDs the user actually named — accounts commonly carry many TB of
  entitlements, well beyond available disk.
- One ID can be many files; a corpus of 20+ parts and hundreds of GB is normal. Check the
  sizes in `corpora.tsv` before starting something huge.
- Never print `ldc.env`, `cookies.txt`, or `$LDC_PASSWORD`.
- Re-running is safe: verified files are skipped, partial ones resume.
