<div align="center">

  <a href="https://img.shields.io/github/actions/workflow/status/llmfy-labs/llmfy-guardrails-python/release.yml">![llmfy-guardrails](https://img.shields.io/github/actions/workflow/status/llmfy-labs/llmfy-guardrails-python/release.yml?style=for-the-badge&logo=pypi&logoColor=blue&label=publish
  )</a>
  <a href="https://pypi.org/project/llmfy-guardrails/0.1.0">![llmfy-guardrails](https://img.shields.io/badge/llmfy--guardrails-v0.1.0-31CA9C.svg?style=for-the-badge&logo=pypi&logoColor=yellow)</a>
  <a href="https://pypi.org/project/llmfy-guardrails/">![llmfy-guardrails](https://img.shields.io/pypi/v/llmfy-guardrails?style=for-the-badge&label=latest&labelColor=691DC6&color=B77309)</a>
  <a href="">![python](https://img.shields.io/badge/python->=3.11-4392FF.svg?style=for-the-badge&logo=python&logoColor=4392FF)</a>

</div>

`llmfy-guardrails` is the guardrails plugin for [`llmfy`](https://github.com/llmfy-labs/llmfy-python): `PIIGuard`, a text-based PII detector/masker (`PIIType`, `PIIStrategy`, `PIIDetection`/`PIIDetectionResult`), independent of the LLM/flow layers in the core package.

## How to install

```sh
# Using UV
uv add llmfy-guardrails

# Using pip
pip install llmfy-guardrails
```

Installing `llmfy-guardrails` pulls in `llmfy` (for the shared `LLMfyException` hierarchy) and `pydantic` automatically.

### PII Guard — PERSON_NAME / ADDRESS detection

Most `PIIType`s are regex-based and work with no extra install. `PIIType.PERSON_NAME`/`PIIType.ADDRESS` are the exception — they're backed by an optional spaCy NER model, loaded lazily on first use:

```sh
# Using UV
uv add "llmfy-guardrails[spacy]"
uv add https://github.com/irufano/spacy_ner_pii/releases/download/v0.1.0/xx_ent_pii_sm-0.1.0-py3-none-any.whl

# Using pip
pip install "llmfy-guardrails[spacy]"
pip install https://github.com/irufano/spacy_ner_pii/releases/download/v0.1.0/xx_ent_pii_sm-0.1.0-py3-none-any.whl
```

`xx_ent_pii_sm` isn't published to PyPI, so it can't be pulled in as a normal extra — install it manually from the release wheel above.

Note that `PIIGuard()` defaults to detecting *every* `PIIType`, including these two — pass `exclude_types=[PIIType.PERSON_NAME, PIIType.ADDRESS]` if you don't want this dependency.

## How to use

```python
from llmfy_guardrails import PIIGuard, PIIStrategy, PIIType

guard = PIIGuard(exclude_types=[PIIType.PERSON_NAME, PIIType.ADDRESS])  # strategy=TOKENIZE (default)
result = guard.detect("Contact john.doe@example.com or call (555) 123-4567.")

print(result.processed_text)  # "Contact [EMAIL_1] or call [PHONE_NUMBER_1]."
restored = guard.restore(result.processed_text, result.detections)
print(restored)  # "Contact john.doe@example.com or call (555) 123-4567."
```

See [`llmfy_guardrails/example/pii_example.py`](llmfy_guardrails/example/pii_example.py) for a full walkthrough of every strategy (`TOKENIZE`, `MASK`, `REDACT`, `PARTIAL`), custom types, and the NER-backed types.

## Develop as Contributor

### Commit message format

Commit subjects follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <short summary>

[optional body]
```

- `type` is one of: `feat`, `fix`, `refactor`, `chore`, `ci`, `docs`, `test`.
- For a breaking change, prefix the subject with `[breaking-changes]`, e.g. `[breaking-changes] refactor: consolidate detection logic`.
- Keep the summary in the imperative mood (e.g. "add", not "added"/"adds").
- The release workflow copies each commit's subject and body verbatim into the GitHub release changelog, so write both to be read standalone (see `.github/workflows/release.yml`).

### Version bump rules (automatic tagging)

Every push to `main` is scanned by `.github/workflows/auto-tag.yml`, which tags a new release automatically — no manual `git tag` needed. The bump is decided per commit subject, in this precedence order (highest across all new commits wins):

| Commit subject | Bump |
|---|---|
| `[breaking-changes] <type>: ...` **or** `<type>!: ...` / `<type>(scope)!: ...` | **MAJOR** |
| `feat: ...` (no breaking marker) | **MINOR** |
| `fix: ...` (no breaking marker) | **PATCH** |
| `refactor:`, `chore:`, `ci:`, `docs:`, `test:` alone | no release (bundled into the next qualifying commit) |

A breaking marker always forces MAJOR regardless of type — use it deliberately when a `feat` or `fix` must ship as a major version, e.g. `[breaking-changes] feat: ...` or `feat!: ...`, both equivalent.

### Build package

```sh
uv build
```

### Manual / backfill release

Tagging is automatic (see above). To manually cut or re-run a release, use the `workflow_dispatch` trigger on `release.yml` (GitHub Actions UI → "Release & Publish" → "Run workflow", tag input `vX.Y.Z`) instead of pushing a tag by hand.

### After deploy on local

After CI creates or moves the tag, your local tag ref may be stale. To sync:

```sh
git fetch --tags --force
```

The --force flag is needed because git fetch --tags alone won't update tags that already exist locally.

### Package Development on local

```sh
uv sync --group dev
```

### Check Lints

```sh
uvx ruff check --statistics . 2>&1 | tail -60
```
