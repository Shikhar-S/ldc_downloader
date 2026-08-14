# LDC package downloading tool

Downloads corpora from the Linguistic Data Consortium catalog.

Bash script by Lane Schwartz (dowobeha@gmail.com), based on Python code
(https://github.com/jonmay/ldcdl) by Jonathan May (jonmay@isi.edu).


## Requirements

bash, curl, sed, awk, tr, md5sum, file, and an LDC login.


## Setup

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

`ldc.env` is gitignored. `~/.ldc.env` works too, and environment variables override both.


## Usage

```
$ ./download-ldc-corpora LDC96L14
```

Run with no arguments to refresh `corpora.tsv`, the list of everything your account can
download — column 1 is the corpus ID, column 2 the title, column 6 the filename, size and
MD5:

```
$ ./download-ldc-corpora
$ grep -i celex corpora.tsv | cut -f1,2
LDC96L14	CELEX2
```

Exit codes: `0` ok, `1` config, `2` auth, `3` unknown corpus, `4` disk, `5` download.


## Notes

* Corpora are often split into many files. All parts are downloaded, so check the sizes in
  `corpora.tsv` first — a single corpus can run to hundreds of gigabytes.
* Downloads are verified against the catalog's MD5. Re-running is safe: verified files are
  skipped and interrupted transfers resume.
* Credentials are prompted for only when stdin is a terminal. Otherwise the script exits
  with an error rather than attempting an anonymous login, which makes it safe to run from
  a batch job or an agent.
