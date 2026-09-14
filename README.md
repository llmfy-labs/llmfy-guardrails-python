<div align="center">

  <a href="https://img.shields.io/github/actions/workflow/status/llmfy-labs/llmfy-guardrails-python/release.yml">![llmfy-guardrails](https://img.shields.io/github/actions/workflow/status/llmfy-labs/llmfy-guardrails-python/release.yml?style=for-the-badge&logo=pypi&logoColor=blue&label=publish
  )</a>
  <a href="https://pypi.org/project/llmfy-guardrails/0.1.1">![llmfy-guardrails](https://img.shields.io/badge/llmfy--guardrails-v0.1.1-31CA9C.svg?style=for-the-badge&logo=pypi&logoColor=yellow)</a>
  <a href="https://pypi.org/project/llmfy-guardrails/">![llmfy-guardrails](https://img.shields.io/pypi/v/llmfy-guardrails?style=for-the-badge&label=latest&labelColor=691DC6&color=B77309)</a>
  <a href="">![python](https://img.shields.io/badge/python->=3.11-4392FF.svg?style=for-the-badge&logo=python&logoColor=4392FF)</a>

</div>

`llmfy-guardrails` is a standalone text-based PII detector/masker (`PIIGuard`, `PIIType`, `PIIStrategy`, `PIIDetection`/`PIIDetectionResult`). It has no dependency on [`llmfy`](https://github.com/llmfy-labs/llmfy-python) or any other LLM framework — drop it into any Python pipeline that needs to detect or mask PII in plain text.

## How to install

```sh
# Using UV
uv add llmfy-guardrails

# Using pip
pip install llmfy-guardrails
```

Installing `llmfy-guardrails` pulls in `pydantic` automatically — nothing else.

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

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for commit message format, the automatic version-bump/release process, and local package development commands.
