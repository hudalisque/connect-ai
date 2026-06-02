# Connect-AI Security Review — Verification Task

## Context

We already completed an initial architecture and security review.

Current findings:

### CONFIRMED

* LLM Output → `<run_command>` → `runCommandCaptured()` → `shell:true`

  * Severity: CRITICAL

* `/api/brain-inject`

  * Exists
  * Writes markdown into Brain
  * Triggers refresh and LLM-related flows
  * Severity: HIGH

* `/api/skill-inject`

  * Exists
  * Writes Python tools into agent tool directories
  * Severity: CRITICAL candidate

* Webview → `runTool` → Tool execution path → shell execution chain exists

  * Severity: HIGH (pending further validation)

---

## Objective

Do NOT search for new vulnerabilities.

Do NOT speculate.

Do NOT perform threat modeling.

Only verify the following OPEN items from source code.

Classify each as:

```text
CONFIRMED
NOT PRESENT
PARTIALLY PRESENT
UNCONFIRMED
```

with exact code evidence.

---

# Task 1 — Localhost Binding

Find the actual server startup code.

Search for:

```text
listen(
createServer(
http.createServer(
https.createServer(
```

Determine:

```text
127.0.0.1 only
0.0.0.0
localhost
unspecified binding
```

Provide:

* exact code snippet
* line numbers
* final conclusion

---

# Task 2 — Authentication

Review:

```text
/api/brain-inject
/api/skill-inject
```

Check for:

```text
Authorization
Bearer
Token
API key
x-api-key
secret
signature
```

Determine:

* authentication required?
* authentication optional?
* no authentication?

Provide code evidence.

---

# Task 3 — CORS

Search for:

```text
Access-Control-Allow-Origin
Origin
OPTIONS
CORS
```

Determine:

* restrictive CORS
* wildcard CORS
* no CORS handling

Provide exact code evidence.

---

# Task 4 — Webview CSP

Search for:

```text
Content-Security-Policy
cspSource
nonce
unsafe-inline
unsafe-eval
```

Review Webview HTML generation.

Determine:

* CSP present?
* CSP absent?
* CSP weak?
* CSP strong?

Provide code evidence.

---

# Reporting Format

For each task:

## Finding

### Status

CONFIRMED / NOT PRESENT / PARTIALLY PRESENT / UNCONFIRMED

### Evidence

(code snippet + line numbers)

### Security Assessment

(brief factual assessment only)

### Severity Impact

How this affects existing findings.

---

## Important Rules

1. Do not modify code.
2. Do not suggest fixes yet.
3. Do not search for additional issues.
4. Stay evidence-based.
5. Separate facts from inference.
6. If evidence is missing, explicitly state "UNCONFIRMED".

Goal: close the current OPEN items, not expand scope.
