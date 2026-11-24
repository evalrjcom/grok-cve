<!--
  P0F — Defensive CVE study notes.
  This file is for LEARNING vulnerability classes and remediation ONLY.
  It contains NO exploit code, NO payloads, and NO reproduction steps — by design.
  Code blocks show the *shape* of vulnerable vs fixed code taken from public patches,
  so you can recognize and fix the bug class, not weaponize it.
-->

# 2.8 Insecure Deserialization (CWE-502) — incl. Python pickle

CWE: CWE-502 (https://cwe.mitre.org/data/definitions/502.html)
Root cause: Untrusted bytes are deserialized by a mechanism that can invoke attacker-controlled types/code.
Source → sink: Cookie/queue/file blob → pickle.loads / ObjectInputStream / XStream / BinaryFormatter → object graph with side effects.
Code smells / review questions
- Grep: pickle.loads, yaml.load (unsafe), ObjectInputStream, unserialize(, XStream without allow-list.
- Who can write this blob? Is the type graph allow-listed?
- Smell: signing a blob but still using a powerful deserializer. Never pickle untrusted data.
Fix pattern
- Prefer JSON + schema; Java ObjectInputFilter allow-list; remove unused deserializers.
Vulnerable shape (educational)

Fixed shape

Detection ideas
- Inventory every deserialize call site; trace blob provenance. Focus on removing the unsafe sink.
Real CVE lessons
- [supplementary] CVE-2017-9805 (Apache Struts REST/XStream): XStreamHandler deserialized untrusted XML; KEV 2021-11-03. — https://nvd.nist.gov/vuln/detail/CVE-2017-9805
- [supplementary] CVE-2023-0669 (Fortra GoAnywhere MFT): Pre-auth unsafe License Response Servlet; KEV 2023-02-10. — https://nvd.nist.gov/vuln/detail/CVE-2023-0669
OSCP / report angle
- Name the API and state that any untrusted pickle is a critical design flaw. No gadget chains.

