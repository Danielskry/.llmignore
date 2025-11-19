# .llmignore
A vendor-neutral ignore specification that lets developers describe exactly what Large Language Model (LLM) tooling can read or modify inside a project. Declare secrets, read-only paths, and safe defaults in one file so editors, agents, and automations respect the same boundaries.

- **Version:** v0.1 (draft community specification)

---

## Why `.llmignore` exists
Modern LLM-powered tools can scan entire repositories, index files, summarize code, and submit multi-file edits. Today there is no shared way to mark sensitive areas or forbid automated edits. `.llmignore` provides a single policy file so you can:

- block access to secrets and regulated data  
- prevent automated edits to critical code  
- keep large generated/vendor trees out of prompts  
- express multiple access levels with `[no-access]`, `[read-only]`, and `[default]`

---

## Design principles
- Familiar `.gitignore`-style syntax  
- Lightweight and vendor-neutral  
- Security and privacy focused  
- Optional access-level sections but compatible with plain lists

---

## Quick start

### Basic ignore rules

```gitignore
# Secrets and sensitive files
.env
*.key
*.pem
secrets/*

# Generated artifacts
dist/
build/

# Vendor assets
third_party/**
```

### Access-level sections
```gitignore
[no-access]
secrets/**
db_dumps/*.sql

[read-only]
src/core/math/**

[default]
node_modules/
logs/
```

Use `[no-access]` for anything tools must never read or modify, `[read-only]` for content that may be indexed but not changed, and `[default]` for files the tool should treat conservatively unless explicitly asked.

---

## Specification (v0.1)

### 1. File discovery
- Tools look for `.llmignore` in the project root and subdirectories.
- Rules in deeper directories override parent directories.
- Optional global file locations may also be honored:
  - Unix: `~/.llmignore_global`
  - Windows: `%USERPROFILE%\.llmignore_global`

### 2. Syntax
`.llmignore` follows `.gitignore` semantics:

- `#` starts a comment; blank lines are ignored.  
- `*` matches any characters except `/`, `**` spans directories.  
- A trailing `/` targets directories.  
- Patterns without a leading `/` apply recursively.  
- Patterns starting with `/` are relative to the file's location.  
- `!pattern` negates a rule within the same section.

### 3. Sections and precedence
Optional INI-style headers define access levels: `[no-access]`, `[read-only]`, `[default]`.  
Lines before the first header implicitly belong to `[default]`.

If multiple sections match a path, precedence is:
1. `no-access`
2. `read-only`
3. `default`
4. unmatched

Negations only apply within the section where they appear.

### 4. Access-level semantics

**[no-access]**
- Must not be read, indexed, or sent to an LLM.
- Must not be modified automatically; only explicit user overrides apply.

**[read-only]**
- May be read and indexed.
- Must not be modified automatically.

**[default]**
- Should be treated conservatively.
- Should not be modified without user approval.
- May be used when explicitly referenced by the user.

### 5. Interaction with other ignore files
Tools may optionally consider `.gitignore`, `.dockerignore`, `.cursorignore`, or vendor-specific files, but `.llmignore` rules always take precedence for LLM behavior.

### 6. Versioning
- Uses `major.minor` semantics.  
- Major bumps are breaking; minor bumps are additive.  
- Unknown future directives should be handled conservatively.

---

## Examples

### Multi-level configuration
```gitignore
# Part of [default]
*.log

[no-access]
secrets/**
finance/*.xlsx

[read-only]
src/core/**
docs/**/*.md

[default]
node_modules/
dist/

!dist/README.md  # Negated rule
```
`dist/README.md` is explicitly allowed even though `dist/` is ignored, while everything under `secrets/` remains fully blocked.
