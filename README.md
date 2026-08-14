# LDC package downloading tool

Bash script by Lane Schwartz (dowobeha@gmail.com)

based on Python code (https://github.com/jonmay/ldcdl) by Jonathan May (jonmay@isi.edu)

04 March 2016


# Requirements:

* bash
* curl
* sed, awk, tr
* md5sum, file
* LDC login/password


# Setup

Put your credentials and download location in an env file:

```
$ cp ldc.env.example ldc.env
$ $EDITOR ldc.env
$ chmod 600 ldc.env
```

```sh
LDC_EMAIL='you@somewhere.edu'
LDC_PASSWORD='your-ldc-password'
LDC_DOWNLOAD_DIR='/path/to/ldc_data'
```

`ldc.env` is gitignored. `~/.ldc.env` works too, and real environment variables take
precedence over both, so you can override any of them for a single run:

```
$ LDC_DOWNLOAD_DIR=/scratch ./download-ldc-corpora LDC96L14
```

If no credentials are configured and you are at a terminal, the script prompts for them as
it always did. If there is no terminal — a batch job, or an agent driving it — it exits
with a clear error instead of silently attempting an anonymous login.


# Usage:

```
$ ./download-ldc-corpora LDC96L14
[2026-08-14 02:31:44 EDT] Using configuration from /path/to/ldc_downloader/ldc.env
[2026-08-14 02:31:44 EDT] Download directory: /path/to/ldc_data
[2026-08-14 02:31:45 EDT] Accessing LDC login page
[2026-08-14 02:31:45 EDT] Logging in to your LDC account
[2026-08-14 02:31:46 EDT] Accessing list of your LDC corpora
[2026-08-14 02:31:46 EDT] LDC96L14 (CELEX2): 1 part(s), 1 to fetch, 0.07 GB
[2026-08-14 02:31:48 EDT] Downloading LDC96L14.tgz
[2026-08-14 02:31:51 EDT] Saved LDC96L14__CELEX2__LDC96L14.tgz
[2026-08-14 02:31:51 EDT] All requested corpora are present in /path/to/ldc_data
```

Run with no arguments to refresh the list of corpora your account can download; it is written
to `corpora.tsv` in the download directory, one row per file, with the corpus ID in column 1,
the title in column 2, and the filename, size and MD5 in column 6.

```
$ ./download-ldc-corpora
$ grep -i celex corpora.tsv | cut -f1,2
LDC96L14	CELEX2
```

Subsequent runs reuse the session cached in `cookies.txt`, so they normally skip the login
entirely.

Exit codes: `0` success, `1` configuration, `2` authentication, `3` corpus not available to
your account, `4` not enough disk space, `5` download or verification failure.


# Notes

* **Corpora are often split into several files.** The catalog lists one row per part, so a
  single ID can map to twenty or more files and hundreds of gigabytes. All parts are
  downloaded. Check the sizes in `corpora.tsv` before starting a large one.
* **The same file can be listed twice** under different download URLs when an account has
  been granted it more than once. Duplicates are detected by filename and MD5 and fetched
  once.
* **Downloads are verified** against the MD5 the catalog publishes. A few corpora show
  `MD5 Checksum: calculating` instead; those are verified against the exact byte count the
  server reports.
* **Re-running is safe.** A file that is already present and matches its checksum is skipped
  without contacting the server, and an interrupted transfer resumes where it left off.
  Files in progress are named `*.part` and are renamed only once verified.
* **Most catalog filenames have no extension.** The real one is taken from the server's
  `Content-Disposition` header, so files are not blindly named `.tgz`.
* Your password is passed to curl through a private temporary file rather than on the command
  line, so it does not show up in `ps` on a shared machine.
* LDC counts every download server-side; the count is column 4 of `corpora.tsv`. It is a
  counter, not a quota.


# Known Issues

* The catalog is scraped from HTML, so a redesign of the LDC downloads page will break
  parsing. The script checks that what it parsed looks like a corpus table and fails loudly
  rather than writing a bad `corpora.tsv`.
