# P0F — Defensive CVE Study

A study resource for OSCP preparation and defensive bug-bounty growth, built from a vulnerability-class cheatsheet.

## No-exploit policy

**This folder contains no exploit code, no payloads, and no reproduction steps — by design.**
Every file explains a vulnerability so you can *recognize, fix, and detect* it:

- plain-English root cause and source-to-sink explanation
- the **shape** of vulnerable vs fixed code, drawn from the real public patch
- detection signals (log and behaviour indicators, not attack strings)
- remediation and the OSCP / bug-report learning angle
- links to NVD, the vendor advisory, and the fix commit

Where a vendor is closed-source and published no diff, the file gives the vendor remediation and says so.

## Contents

### `00-primer/` — How to read a CVE; the coverage map

- [1. How to Read a CVE (primer)](00-primer/1-how-to-read-a-cve-primer.md)
- [1b. Coverage map](00-primer/1b-coverage-map.md)

### `01-bug-classes-core/` — Core bug classes (taught in lessons): command injection, SQLi, buffer overflow, LFI/RFI, path traversal, file upload, auth bypass, deserialization, SSRF, XXE, XSS, IDOR, races, UAF, CSRF, JWT/crypto, prototype pollution

- [2.1 OS Command Injection (CWE-78)](01-bug-classes-core/2-1-os-command-injection-cwe-78.md)
- [2.10 SSRF (CWE-918)](01-bug-classes-core/2-10-ssrf-cwe-918.md)
- [2.11 XXE (CWE-611)](01-bug-classes-core/2-11-xxe-cwe-611.md)
- [2.12 Cross-Site Scripting / XSS (CWE-79)](01-bug-classes-core/2-12-cross-site-scripting-xss-cwe-79.md)
- [2.13 IDOR / Broken Access Control (CWE-639, 862, 863)](01-bug-classes-core/2-13-idor-broken-access-control-cwe-639-862-863.md)
- [2.14 Race Conditions / TOCTOU (CWE-362, 367)](01-bug-classes-core/2-14-race-conditions-toctou-cwe-362-367.md)
- [2.15 Use-After-Free (CWE-416)](01-bug-classes-core/2-15-use-after-free-cwe-416.md)
- [2.16 Integer Overflow / Truncation / OOB from size math (CWE-190, 195, 197, 787)](01-bug-classes-core/2-16-integer-overflow-truncation-oob-from-size-math-cwe-190-195-197-78.md)
- [2.17 CSRF (CWE-352)](01-bug-classes-core/2-17-csrf-cwe-352.md)
- [2.18 JWT / Crypto Verification (CWE-347, 327)](01-bug-classes-core/2-18-jwt-crypto-verification-cwe-347-327.md)
- [2.19 Prototype Pollution (CWE-1321)](01-bug-classes-core/2-19-prototype-pollution-cwe-1321.md)
- [2.2 SQL Injection (CWE-89)](01-bug-classes-core/2-2-sql-injection-cwe-89.md)
- [2.3 Stack Buffer Overflow (CWE-121)](01-bug-classes-core/2-3-stack-buffer-overflow-cwe-121.md)
- [2.4 File Inclusion (CWE-98)](01-bug-classes-core/2-4-file-inclusion-cwe-98.md)
- [2.5 Path Traversal (CWE-22)](01-bug-classes-core/2-5-path-traversal-cwe-22.md)
- [2.6 Unrestricted File Upload (CWE-434)](01-bug-classes-core/2-6-unrestricted-file-upload-cwe-434.md)
- [2.7 Authentication Bypass (CWE-287)](01-bug-classes-core/2-7-authentication-bypass-cwe-287.md)
- [2.8 Insecure Deserialization (CWE-502) — incl. Python pickle](01-bug-classes-core/2-8-insecure-deserialization-cwe-502-incl-python-pickle.md)
- [2.9 Expression-Language Injection (CWE-917)](01-bug-classes-core/2-9-expression-language-injection-cwe-917.md)
- [2. Bug Classes — Covered in Lessons](01-bug-classes-core/2-bug-classes-covered-in-lessons.md)

### `02-extended-classes/` — Extended classes with CVE walks: open redirect, session fixation, CORS, request smuggling, SSTI, NoSQL injection, LDAP injection, mass assignment, business logic, clickjacking, hard-coded creds, missing auth, info exposure, type confusion, double free, format string, null deref, privesc misconfig, cache poisoning

- [3.1 Open Redirect (CWE-601)](02-extended-classes/3-1-open-redirect-cwe-601.md)
- [3.10 Clickjacking / UI Redress (CWE-1021)](02-extended-classes/3-10-clickjacking-ui-redress-cwe-1021.md)
- [3.11 Hard-Coded Credentials (CWE-798)](02-extended-classes/3-11-hard-coded-credentials-cwe-798.md)
- [3.12 Missing Authentication for Critical Function (CWE-306)](02-extended-classes/3-12-missing-authentication-for-critical-function-cwe-306.md)
- [3.13 Information Exposure (CWE-200)](02-extended-classes/3-13-information-exposure-cwe-200.md)
- [3.14 Type Confusion (CWE-843)](02-extended-classes/3-14-type-confusion-cwe-843.md)
- [3.15 Double Free (CWE-415)](02-extended-classes/3-15-double-free-cwe-415.md)
- [3.16 Format String (CWE-134)](02-extended-classes/3-16-format-string-cwe-134.md)
- [3.17 NULL Pointer Dereference (CWE-476)](02-extended-classes/3-17-null-pointer-dereference-cwe-476.md)
- [3.18 sudo / SUID / Windows Privilege-Escalation Misconfigurations (CWE-250, 269)](02-extended-classes/3-18-sudo-suid-windows-privilege-escalation-misconfigurations-cwe-250-.md)
- [3.19 Cache Poisoning / Deception (CWE-349, 525)](02-extended-classes/3-19-cache-poisoning-deception-cwe-349-525.md)
- [3.2 Session Fixation (CWE-384)](02-extended-classes/3-2-session-fixation-cwe-384.md)
- [3.3 CORS Misconfiguration (CWE-942)](02-extended-classes/3-3-cors-misconfiguration-cwe-942.md)
- [3.4 HTTP Request Smuggling (CWE-444)](02-extended-classes/3-4-http-request-smuggling-cwe-444.md)
- [3.5 Server-Side Template Injection (CWE-1336)](02-extended-classes/3-5-server-side-template-injection-cwe-1336.md)
- [3.6 NoSQL Injection (CWE-943)](02-extended-classes/3-6-nosql-injection-cwe-943.md)
- [3.7 LDAP Injection (CWE-90)](02-extended-classes/3-7-ldap-injection-cwe-90.md)
- [3.8 Mass Assignment (CWE-915)](02-extended-classes/3-8-mass-assignment-cwe-915.md)
- [3.9 Business-Logic Flaws (CWE-840)](02-extended-classes/3-9-business-logic-flaws-cwe-840.md)
- [3. Extended Bug Classes — CVE Walks (supplementary)](02-extended-classes/3-extended-bug-classes-cve-walks-supplementary.md)

### `03-additional-categories/` — Additional categories: unquoted service path, DLL hijack, weak permissions, symlink following, ReDoS, CRLF injection, GraphQL, weak password reset, weak randomness, cert validation, XPath injection, error-message leaks, secrets in logs, CSV injection

- [2b.1 Unquoted Windows Service Path (CWE-428)](03-additional-categories/2b-1-unquoted-windows-service-path-cwe-428.md)
- [2b.10 Improper Certificate Validation (CWE-295)](03-additional-categories/2b-10-improper-certificate-validation-cwe-295.md)
- [2b.11 XPath Injection (CWE-643)](03-additional-categories/2b-11-xpath-injection-cwe-643.md)
- [2b.12 Sensitive Data in Error Messages (CWE-209)](03-additional-categories/2b-12-sensitive-data-in-error-messages-cwe-209.md)
- [2b.13 Secrets in Log Files (CWE-532)](03-additional-categories/2b-13-secrets-in-log-files-cwe-532.md)
- [2b.14 CSV / Formula Injection (CWE-1236)](03-additional-categories/2b-14-csv-formula-injection-cwe-1236.md)
- [2b.2 DLL Search-Order Hijacking (CWE-427)](03-additional-categories/2b-2-dll-search-order-hijacking-cwe-427.md)
- [2b.3 Incorrect Default Permissions (CWE-732, CWE-276)](03-additional-categories/2b-3-incorrect-default-permissions-cwe-732-cwe-276.md)
- [2b.4 Symlink / Link Following (CWE-59)](03-additional-categories/2b-4-symlink-link-following-cwe-59.md)
- [2b.5 Regular Expression DoS (ReDoS) (CWE-1333)](03-additional-categories/2b-5-regular-expression-dos-redos-cwe-1333.md)
- [2b.6 CRLF / HTTP Header Injection (CWE-113, CWE-93)](03-additional-categories/2b-6-crlf-http-header-injection-cwe-113-cwe-93.md)
- [2b.7 GraphQL Introspection and Batching](03-additional-categories/2b-7-graphql-introspection-and-batching.md)
- [2b.8 Weak Password Reset (CWE-640)](03-additional-categories/2b-8-weak-password-reset-cwe-640.md)
- [2b.9 Weak Randomness / Insufficient Entropy (CWE-330)](03-additional-categories/2b-9-weak-randomness-insufficient-entropy-cwe-330.md)
- [2b. Additional Vulnerability Categories](03-additional-categories/2b-additional-vulnerability-categories.md)

### `05-realworld-walks/` — Real-world CVE walks (batch 1): Spring4Shell, Text4Shell, Jenkins, Grafana, GitLab+ExifTool, PHP-CGI, xz, regreSSHion, Dirty Pipe, sudo, Next.js, Django, tj-actions, Erlang/OTP

- [3b. More CVEs to Study — Real-World Walks (supplementary)](05-realworld-walks/3b-more-cves-to-study-real-world-walks-supplementary.md)
- [CVE walk — CVE-2021-22205 (GitLab CE/EE 11.9 – < 13.10.3 — uploaded images reach ExifTool; root cause CVE-2021-22204 in ExifTool < 12.24; KEV)](05-realworld-walks/cve-2021-22205-cve-walk-cve-2021-22205-gitlab-ce-ee-11-9-13-10-3-uploa.md)
- [CVE walk — CVE-2021-3156 (sudo 1.8.2–1.8.31p2, 1.9.0–1.9.5p1, plus 1.7.7–1.7.10p9 per the vendor — heap overflow while unescaping arguments; KEV)](05-realworld-walks/cve-2021-3156-cve-walk-cve-2021-3156-sudo-1-8-2-1-8-31p2-1-9-0-1-9-5p1.md)
- [CVE walk — CVE-2021-43798 (Grafana 8.0.0-beta1 – 8.3.0 — plugin asset path cleaned relative to the plugin directory; KEV since 2025)](05-realworld-walks/cve-2021-43798-cve-walk-cve-2021-43798-grafana-8-0-0-beta1-8-3-0-plugi.md)
- [CVE walk — CVE-2022-0847 (Linux kernel 5.8 – < 5.16.11 / 5.15.25 / 5.10.102 — pipe_buffer "flags" left uninitialised; KEV)](05-realworld-walks/cve-2022-0847-cve-walk-cve-2022-0847-linux-kernel-5-8-5-16-11-5-15-25-.md)
- [CVE walk — CVE-2022-22965 (Spring Framework 5.3.0–5.3.17, 5.2.0–5.2.19 and older — data-binding filter keyed on names, not types; KEV)](05-realworld-walks/cve-2022-22965-cve-walk-cve-2022-22965-spring-framework-5-3-0-5-3-17-5.md)
- [CVE walk — CVE-2022-34265 (Django 3.2 < 3.2.14, 4.0 < 4.0.6 — Trunc(kind) and Extract(lookup_name) placed into SQL)](05-realworld-walks/cve-2022-34265-cve-walk-cve-2022-34265-django-3-2-3-2-14-4-0-4-0-6-tru.md)
- [CVE walk — CVE-2022-42889 (Apache Commons Text 1.5–1.9 — risky lookups enabled by default in the interpolator)](05-realworld-walks/cve-2022-42889-cve-walk-cve-2022-42889-apache-commons-text-1-5-1-9-ris.md)
- [CVE walk — CVE-2024-23897 (Jenkins ≤ 2.441 / LTS ≤ 2.426.2 — CLI argument parser expands @file references; KEV)](05-realworld-walks/cve-2024-23897-cve-walk-cve-2024-23897-jenkins-2-441-lts-2-426-2-cli-a.md)
- [CVE walk — CVE-2024-3094 (XZ Utils 5.6.0 and 5.6.1 — backdoor present in the release tarballs, not the git tree)](05-realworld-walks/cve-2024-3094-cve-walk-cve-2024-3094-xz-utils-5-6-0-and-5-6-1-backdoor.md)
- [CVE walk — CVE-2024-4577 (PHP 8.1 < 8.1.29, 8.2 < 8.2.20, 8.3 < 8.3.8, CGI mode on Windows — Best-Fit character mapping; KEV)](05-realworld-walks/cve-2024-4577-cve-walk-cve-2024-4577-php-8-1-8-1-29-8-2-8-2-20-8-3-8-3.md)
- [CVE walk — CVE-2024-6387 (OpenSSH sshd 8.5p1–9.7p1 — unsafe work inside the login-grace-time signal handler)](05-realworld-walks/cve-2024-6387-cve-walk-cve-2024-6387-openssh-sshd-8-5p1-9-7p1-unsafe-w.md)
- [CVE walk — CVE-2025-29927 (Next.js < 12.3.5 / 13.5.9 / 14.2.25 / 15.2.3 — internal "skip middleware" header honored from outside)](05-realworld-walks/cve-2025-29927-cve-walk-cve-2025-29927-next-js-12-3-5-13-5-9-14-2-25-1.md)
- [CVE walk — CVE-2025-30066 (tj-actions/changed-files ≤ 45.0.7 — version tags repointed to a malicious commit; KEV)](05-realworld-walks/cve-2025-30066-cve-walk-cve-2025-30066-tj-actions-changed-files-45-0-7.md)
- [CVE walk — CVE-2025-32433 (Erlang/OTP SSH server < 27.3.3 / 26.2.5.11 / 25.3.2.20 — connection-layer messages processed before authentication; KEV)](05-realworld-walks/cve-2025-32433-cve-walk-cve-2025-32433-erlang-otp-ssh-server-27-3-3-26.md)

### `06-realworld-walks-batch3/` — Real-world CVE walks (batch 3): Struts, httpd SSRF, ProxyShell, Magento XXE, ActiveMQ, Ivanti, Confluence, Citrix Bleed, Looney Tunables, Fortinet, Python tarfile, Rails, runc, HTTP/2 Rapid Reset, PAN-OS

- [3c. More CVEs to Study (batch 3) — Real-World Walks (supplementary)](06-realworld-walks-batch3/3c-more-cves-to-study-batch-3-real-world-walks-supplementary.md)
- [CVE walk — CVE-2007-4559 (Python tarfile — extract / extractall path confinement; PEP 706 filters)](06-realworld-walks-batch3/cve-2007-4559-cve-walk-cve-2007-4559-python-tarfile-extract-extractall.md)
- [CVE walk — CVE-2017-5638 (Apache Struts 2.3.x < 2.3.32, 2.5.x < 2.5.10.1 — Jakarta multipart error messages; KEV)](06-realworld-walks-batch3/cve-2017-5638-cve-walk-cve-2017-5638-apache-struts-2-3-x-2-3-32-2-5-x-.md)
- [CVE walk — CVE-2018-13379 (Fortinet FortiOS SSL VPN web portal — unauthenticated path traversal; KEV)](06-realworld-walks-batch3/cve-2018-13379-cve-walk-cve-2018-13379-fortinet-fortios-ssl-vpn-web-po.md)
- [CVE walk — CVE-2019-5418 (Ruby on Rails Action View < 5.2.2.1 / 5.1.6.2 / 5.0.7.2 / 4.2.11.1 — Accept header file disclosure; KEV)](06-realworld-walks-batch3/cve-2019-5418-cve-walk-cve-2019-5418-ruby-on-rails-action-view-5-2-2-1.md)
- [CVE walk — CVE-2021-34473 (Microsoft Exchange Server 2013 / 2016 / 2019 — pre-auth access to a backend resource; KEV)](06-realworld-walks-batch3/cve-2021-34473-cve-walk-cve-2021-34473-microsoft-exchange-server-2013-.md)
- [CVE walk — CVE-2021-40438 (Apache HTTP Server ≤ 2.4.48 — mod_proxy SSRF; KEV)](06-realworld-walks-batch3/cve-2021-40438-cve-walk-cve-2021-40438-apache-http-server-2-4-48-mod-p.md)
- [CVE walk — CVE-2023-22515 (Atlassian Confluence Data Center / Server 8.0.0–8.5.1 — broken access control, unauth admin creation; KEV)](06-realworld-walks-batch3/cve-2023-22515-cve-walk-cve-2023-22515-atlassian-confluence-data-cente.md)
- [CVE walk — CVE-2023-44487 (HTTP/2 — Rapid Reset / stream-cancel flood; KEV)](06-realworld-walks-batch3/cve-2023-44487-cve-walk-cve-2023-44487-http-2-rapid-reset-stream-cance.md)
- [CVE walk — CVE-2023-46604 (Apache ActiveMQ OpenWire — class name in the protocol; KEV)](06-realworld-walks-batch3/cve-2023-46604-cve-walk-cve-2023-46604-apache-activemq-openwire-class-.md)
- [CVE walk — CVE-2023-46805 (Ivanti Connect Secure 9.x / 22.x and Policy Secure — web-component auth bypass; KEV)](06-realworld-walks-batch3/cve-2023-46805-cve-walk-cve-2023-46805-ivanti-connect-secure-9-x-22-x-.md)
- [CVE walk — CVE-2023-4911 (GNU C Library ld.so — GLIBC_TUNABLES buffer overflow, "Looney Tunables"; KEV)](06-realworld-walks-batch3/cve-2023-4911-cve-walk-cve-2023-4911-gnu-c-library-ld-so-glibc-tunable.md)
- [CVE walk — CVE-2023-4966 (Citrix / NetScaler ADC and Gateway — session-token disclosure, "Citrix Bleed"; KEV)](06-realworld-walks-batch3/cve-2023-4966-cve-walk-cve-2023-4966-citrix-netscaler-adc-and-gateway-.md)
- [CVE walk — CVE-2024-21626 (runc ≤ 1.1.11 — leaked host file descriptors and an unchecked working directory)](06-realworld-walks-batch3/cve-2024-21626-cve-walk-cve-2024-21626-runc-1-1-11-leaked-host-file-de.md)
- [CVE walk — CVE-2024-3400 (Palo Alto Networks PAN-OS GlobalProtect — unauthenticated file creation to command injection; KEV)](06-realworld-walks-batch3/cve-2024-3400-cve-walk-cve-2024-3400-palo-alto-networks-pan-os-globalp.md)
- [CVE walk — CVE-2024-34102 (Adobe Commerce / Magento Open Source — unauthenticated XXE, "CosmicSting"; KEV)](06-realworld-walks-batch3/cve-2024-34102-cve-walk-cve-2024-34102-adobe-commerce-magento-open-sou.md)

### `07-new-category-walks/` — New-category CVE walks: one per additional category (Cloudflare WARP, Cisco AnyConnect, Docker Desktop, Archive_Tar, moment.js, Netdata, SuiteCRM, GitLab reset, Zerologon, CurveBall, XML::Sig, Auth0 WCF, Jupyter, Nikto)

- [3d. New-category CVE walks (supplementary) (overview)](07-new-category-walks/00-3d-new-category-cve-walks-supplementary-intro.md)
- [CVE walk — CVE-2018-11652 (Nikto 2.1.6 and earlier — unsanitised fields in the CSV report)](07-new-category-walks/cve-2018-11652-cve-walk-cve-2018-11652-nikto-2-1-6-and-earlier-unsanit.md)
- [CVE walk — CVE-2018-18837 (Netdata 1.10.0 — header injection via filename-style parameters)](07-new-category-walks/cve-2018-18837-cve-walk-cve-2018-18837-netdata-1-10-0-header-injection.md)
- [CVE walk — CVE-2019-15752 (Docker Desktop Community Edition — writable helper directory)](07-new-category-walks/cve-2019-15752-cve-walk-cve-2019-15752-docker-desktop-community-editio.md)
- [CVE walk — CVE-2019-7644 (Auth0-WCF-Service-JWT before 1.0.4 — expected signature in the error message)](07-new-category-walks/cve-2019-7644-cve-walk-cve-2019-7644-auth0-wcf-service-jwt-before-1-0-.md)
- [CVE walk — CVE-2020-0601 (Windows CryptoAPI / "CurveBall" — ECC group parameters not pinned)](07-new-category-walks/cve-2020-0601-cve-walk-cve-2020-0601-windows-cryptoapi-curveball-ecc-g.md)
- [CVE walk — CVE-2020-1472 (Windows Netlogon / "Zerologon" — insufficient credential entropy)](07-new-category-walks/cve-2020-1472-cve-walk-cve-2020-1472-windows-netlogon-zerologon-insuff.md)
- [CVE walk — CVE-2020-3433 (Cisco AnyConnect for Windows — DLL search path via IPC)](07-new-category-walks/cve-2020-3433-cve-walk-cve-2020-3433-cisco-anyconnect-for-windows-dll-.md)
- [CVE walk — CVE-2020-36193 (PEAR Archive_Tar — symlink target not kept inside the extract path)](07-new-category-walks/cve-2020-36193-cve-walk-cve-2020-36193-pear-archive-tar-symlink-target.md)
- [CVE walk — CVE-2022-2147 (Cloudflare WARP for Windows — unquoted service path)](07-new-category-walks/cve-2022-2147-cve-walk-cve-2022-2147-cloudflare-warp-for-windows-unquo.md)
- [CVE walk — CVE-2022-24757 (Jupyter Server before 1.15.4 — auth cookie written to 5xx logs)](07-new-category-walks/cve-2022-24757-cve-walk-cve-2022-24757-jupyter-server-before-1-15-4-au.md)
- [CVE walk — CVE-2022-31129 (moment.js — quadratic RFC 2822 comment stripping)](07-new-category-walks/cve-2022-31129-cve-walk-cve-2022-31129-moment-js-quadratic-rfc-2822-co.md)
- [CVE walk — CVE-2023-47643 (SuiteCRM before 8.4.2 — unauthenticated GraphQL introspection)](07-new-category-walks/cve-2023-47643-cve-walk-cve-2023-47643-suitecrm-before-8-4-2-unauthent.md)
- [CVE walk — CVE-2023-7028 (GitLab CE/EE — password-reset mail to an unverified address)](07-new-category-walks/cve-2023-7028-cve-walk-cve-2023-7028-gitlab-ce-ee-password-reset-mail-.md)
- [CVE walk — CVE-2026-9390 (Perl XML::Sig before 0.71 — XPath built from a Reference URI)](07-new-category-walks/cve-2026-9390-cve-walk-cve-2026-9390-perl-xml-sig-before-0-71-xpath-bu.md)

### `08-reference/` — Master "what to look for" checklist, CVE quick-reference table, responsible-disclosure scope

- [4. Master "What to Look For" Checklist](08-reference/4-master-what-to-look-for-checklist.md)
- [5. CVE Quick-Reference Table](08-reference/5-cve-quick-reference-table.md)
- [6. Responsible Disclosure & Bug Bounty Scope](08-reference/6-responsible-disclosure-bug-bounty-scope.md)

