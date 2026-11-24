<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.9 Expression-Language Injection (CWE-917)

CWE: CWE-917 (https://cwe.mitre.org/data/definitions/917.html)
Root cause: Untrusted input is evaluated as an expression language (OGNL, SpEL, JNDI lookup strings, Freemarker).
Source → sink: Request/header/log message → template/EL evaluator / JNDI resolve → method invocation.
Code smells / review questions
- Grep: OgnlUtil, SpelExpressionParser, expression-language delimiters in server templates fed by users.
- Is user input the expression, or only data bound into a compiled template?
- Smell: logging frameworks resolving lookups in message content.
Fix pattern
- Never evaluate user strings as EL; disable JNDI/message lookups; sandbox templates.
Detection ideas
- Hunt formula/filter/dynamic-template features; review log configs.
Real CVE lessons
- [supplementary] CVE-2021-44228 (Apache Log4j2): JNDI features in log messages; CWE-917 among others; KEV 2021-12-10. — https://nvd.nist.gov/vuln/detail/CVE-2021-44228
- [supplementary] CVE-2022-26134 (Atlassian Confluence): OGNL injection, unauthenticated RCE; KEV 2022-06-02. — https://nvd.nist.gov/vuln/detail/CVE-2022-26134
- [supplementary] CVE-2017-12611 (Apache Struts Freemarker): Unintentional expression in Freemarker tag. — https://nvd.nist.gov/vuln/detail/CVE-2017-12611
OSCP / report angle
- Describe the evaluator and untrusted string path. No OGNL/JNDI payload strings in the report.

