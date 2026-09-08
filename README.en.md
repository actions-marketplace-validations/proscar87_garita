# Garita

[Español](https://github.com/proscar87/garita/blob/main/README.md) · **English**

**Keeps personal data and credentials out of your repository.** Names from
your own records, official IDs from 16 countries across the Americas and the
Iberian peninsula (CURP, RFC, CLABE, CPF, CUIT, RUT, DNI, IBAN…), phone
numbers and secrets — as a `pre-commit` hook and as a GitHub Action.

## Why it exists

It came out of a common problem: a project where the financial records **had
to be** versioned — someone will audit them — and the roster of people could
**not** be. Privacy law regulates it and git history doesn't forget.

The rule that emerged sums up the tool:

> **The line is the unit, not the name.**
> You may version unit 47 and what it owes. Never the link between that unit
> and the person living in it.

Nothing existed to enforce it. Microsoft Presidio covers 20+ countries and
**Mexico isn't one**. GitGuardian **explicitly rejects** PII patterns.
GitHub charges for custom patterns, and they're secret patterns anyway — not
name lists. `git-secrets`, the only tool that could ban arbitrary strings, has
been unmaintained since 2019.

## The core idea

A guard that compares against a list written in its own config has a fatal
flaw: **the list is itself personal data**. To keep the roster out of the
repo, you'd have to write the roster into the repo.

The way out: most projects handling real data already have a **synthetic data
generator** for tests and seeds — and it already knows which real names it
must never produce by accident.

```yaml
nombres:
  - scripts/generate_fake_data.py:FORBIDDEN
```

Garita reads it from there **via AST, without executing the file**. One list,
impossible to desynchronise.

## What makes it different

**Mexican national IDs with checksum validation.** A detector that flags any
18-character string as a CURP cries wolf constantly — and a guard that cries
wolf gets ignored. With checksum validation, false positives drop by 90× to
5,000× depending on the identifier.

## Install

```yaml
# .pre-commit-config.yaml — start here
repos:
  - repo: https://github.com/proscar87/garita
    rev: v1.0.0
    hooks:
      - id: garita
```

```yaml
# .github/workflows/garita.yml — the backstop
- uses: actions/checkout@v4
- uses: proscar87/garita@v1
```

Hook first: if your only check is in CI, by the time it fails the data already
lives in a commit — and the fix goes from "delete a line" to "rewrite history
and notify everyone who cloned".

**Existing repo with prior findings?** Run `garita --linea-base` to freeze
them as accepted debt: CI then fails only on *new* findings, while old ones
stay visible in the report. The baseline file stores only counts per file and
detector — no values, no hashes — so it's safe to commit.

**And the past counts too:** `garita --historial` audits every version of
every file that ever passed through the repo — the secret committed three
months ago and "deleted" the next day is invisible to a normal scan, but git
never forgets. The report separates what's still in the tree from what lives
only in history, where the fix is rotating the credential — never a silent
history rewrite.

**A note on vocabulary**, since the config keys are in Spanish: `exenciones`
are what other tools call *exceptions*, *ignores* or an *allowlist* — each
one needs a written `motivo` (reason), and Garita refuses to run without it.
`nombres` is the list of names to protect, `clientes` the client list,
`paises` the country selection, `detectores` the on/off switches. Run
`garita --proponer-exenciones` and it writes the block for you, with the
reason left blank for you to fill in.

Config, findings format and design rationale: see the
[Spanish README](https://github.com/proscar87/garita/blob/main/README.md) and [`docs/DISENO.md`](https://github.com/proscar87/garita/blob/main/docs/DISENO.md). Built with
[Claude Code](https://claude.com/claude-code). MIT.
