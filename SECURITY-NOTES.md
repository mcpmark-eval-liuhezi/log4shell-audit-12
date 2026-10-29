# Log4Shell (CVE-2021-44228) — Security Notes

Briefing notes for the quarterly security review. All facts below are taken from the public sources cited at the bottom of this file; versions and scores are not from memory.

## Core facts

| Field | Value |
| --- | --- |
| CVE identifier | **CVE-2021-44228** |
| What it is | Apache Log4j 2 remote code execution via JNDI lookups. An attacker who can control log messages or log message parameters can trigger compilation and loading of a remotely supplied Java class from an LDAP/RMI server, via the `${jndi:...}` lookup. Only the `log4j-core` JAR is affected; an app built with only `log4j-api` is not. (Fix for related Log4j 1 issue CVE-2021-4104, which targets Log4j 1 JMSAppender.) |
| Affected Log4j 2 versions | **`2.0-beta9` through `2.14.1`**, i.e. `[2.0-beta9, 2.3.1)`, `[2.4, 2.12.2)`, and `[2.13.0, 2.15.0)` |
| Fixed versions | `2.15.0` for Java 8 and later; `2.12.2` for Java 7; `2.3.1` for Java 6. JNDI lookups are disabled by default from `2.16`; current recommended production floor is **`2.17.1` or higher** |
| Severity | **CVSS 3.1 score: 10.0 CRITICAL** (CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H) — the highest possible severity |

## Remediation at a glance

1. **Upgrade** `log4j-core` to the production floor `2.17.1+` (or the closest fix version on the affected Java line); keep `log4j-api` at the same version. See issue #2.
2. **Detect vulnerable deployments** — scan for the `JndiLookup.class` (e.g. `.../org/apache/logging/log4j/core/lookup/JndiLookup.class`) and `log4j-core-*.jar` binaries, and search logs for `${jndi:` strings and the class signatures observed on exploitation. See issue #3.
3. **Harden without an immediate upgrade** — set `LOG4J_FORMAT_MSG_NO_LOOKUPS=true` (or `Dlog4j2.formatMsgNoLookups=true`) and restart the JVM, or remove the `JndiLookup` class from the jar and restart the application. See issue #4.
4. **Automate detection** — run a static scanner (e.g. OWASP Dependency-Check) in CI so builds fail on vulnerable versions, and publish an SBOM per release. See issue #5.

## Sources

- **Apache Logging Services — Security / CVE-2021-44228 page**: https://logging.apache.org/security.html — affected version ranges, fixed version branches, and the CVSS 10.0 CRITICAL score.
- **NVD — CVE-2021-44228**: https://nvd.nist.gov/vuln/detail/CVE-2021-44228 — CVSS base score and vector, affected/affected version ranges.
- **GitHub Advisory Database — GHSA-jfh8-c2jp-5v3q (Log4Shell)**: https://github.com/advisories/ghsa-jfh8-c2jp-5v3q — patched and backported versions, note that JNDI is disabled by default from 2.16, and general advisory context.
- **NCSC-NL — Detection and mitigation for the Log4j 2 vulnerability** ( Detecting, hunting, and generating mitigations): https://github.com/NCSC-NL/log4shell
- **Microsoft Security Blog — Guidance for preventing, detecting, and hunting for the Log4j 2 vulnerability**: https://www.microsoft.com/en-us/security/blog/2021/12/11/guidance-for-preventing-detecting-and-hunting-for-cve-2021-44228-log4j-2-exploitation/ — mitigation detection and the `LOG4J_FORMAT_MSG_NO_LOOKUPS` environment variable.
- **Snyk — Log4Shell remediation cheat sheet**: https://snyk.io/blog/log4shell-remediation-cheat-sheet/ — upgrade path, removing the lookup class, and disabling lookups programmatically.
- **JPCERT/CC — Alert Regarding Arbitrary Code Execution Vulnerability (CVE-2021-44228)** (2021-12-11, with January 4, 2022 update): https://www.jpcert.or.jp/english/at/2021/at210050.html — affected versions, updated fix releases (`2.17.1` Java 8+, `2.12.4` Java 7, `2.3.2` Java 6), and exploitation detection signs.

---

Parent tracking issue: [Log4Shell remediation checklist](https://github.com/mcpmark-eval-liuhezi/log4shell-audit-12/issues/1)
