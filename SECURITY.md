# Security Policy

## Public repository scope

This is a public repository. It is designed to be safe to clone, run, and inspect without exposing any real infrastructure.

**What is NOT in this repository:**
- IP addresses, hostnames, or network topology
- Home Assistant entity IDs, tokens, or credentials
- Personal data of any kind
- Camera feeds, location data, or presence information
- Inverter credentials or grid account details
- Any information that could identify or locate the physical installation

**What IS in this repository:**
- Synthetic energy data that mimics real sensor patterns
- Generic environment variable names and documentation
- Architecture decisions and code

## Reporting a vulnerability

If you discover a security issue in HomeOps code or its dependencies, please **do not open a public issue**.

Instead, report it via [GitHub private vulnerability reporting](https://github.com/titusandronicu/homeops/security/advisories/new).

Include:
- A description of the vulnerability
- Steps to reproduce
- Potential impact
- Any suggested fix, if you have one

You will receive a response within 72 hours. Critical issues will be addressed as a priority.

## Dependency scanning

Every pull request and push to `main` is scanned by:
- **Gitleaks** — detects accidentally committed secrets
- **Trivy** — flags `CRITICAL` and `HIGH` CVEs in dependencies and the container image

A CI failure on either of these blocks the merge.

## Home Assistant integration (private local config only)

If you deploy HomeOps against a real Home Assistant instance:

1. Use a **long-lived access token** scoped to the minimum required permissions — read-only access to energy sensors only.
2. Populate `ALLOWED_HA_ENTITIES` with only the specific sensor entity IDs you want HomeOps to read. No wildcards.
3. Never commit your `.env.local` file. It is listed in `.gitignore`.
4. The application enforces the allowlist server-side and will reject any entity ID not explicitly listed.

**Excluded entity domains:** `camera`, `person`, `device_tracker`, `lock`, `alarm_control_panel`, `geo_location`, `proximity`, `zone`.

## AI boundary

The Claude API integration receives only a `RecommendationResult` — a small typed object containing an action, confidence score, and machine-readable reasons. It never receives:
- Raw sensor readings
- User identity or session data
- Home Assistant tokens or configuration
- Network information of any kind

The AI has no write path. It returns text. That text is displayed to the user, who then chooses to Accept or Reject.
