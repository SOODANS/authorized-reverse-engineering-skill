---
name: authorized-software-reverse-engineering
description: Evidence-led, bounded workflow for analyzing authorized software artifacts using static analysis first, with optional controlled dynamic analysis when explicitly authorized.
---

# Authorized Software Reverse Engineering Skill

## Mission

Analyze an authorized software artifact to answer a finite set of behavioral questions. Produce a reproducible, evidence-led report. Do not infer behavior solely from names, strings, imports, or the presence of a library.

## Required inputs

- Artifact path and expected product/version.
- Confirmation that the requester is authorized to inspect the artifact.
- Questions to answer and intended use of the report.
- Operating system and available tools.
- Scope exclusions (for example, static-only; no app execution; no network testing).

If the question is vague, translate it into 3–7 testable hypotheses before analysis.

## Core rules

1. Hash and identify every artifact before analysis.
2. Record tool versions, paths, commands and outputs.
3. Follow data/control flow from source to effect: entry point → processing → storage/serialization → sink → response/error handling.
4. Prefer targeted inspection before whole-artifact disassembly.
5. For native code, map function addresses, call sites, arguments and branches; symbols alone are not proof of behavior.
6. Separate static evidence from runtime observations, analyst inference and unresolved questions.
7. Distinguish an attempted action from a successful outcome. A request call is not proof of server receipt; a local “uploaded” flag is not proof of remote storage.
8. Use only authorized artifacts and tests. Do not bypass access controls, collect unrelated personal data, or conduct intrusive runtime tests outside scope.
9. Never fabricate evidence, command output, file paths, tool versions or citations.
10. Stop when the decision criteria are met or remaining uncertainty requires a different evidence source.

## Workflow

### 1. Scope and identity
- Confirm authorization and limits.
- Record filename, format, product/version, package ID, architecture, SHA-256 and whether all splits/components are present.
- Preserve the original artifact; perform work on a copy.

### 2. Inventory
- Identify manifest/configuration, entry points, permissions, services, receivers, background workers, network clients, storage and native libraries.
- Record tool versions and validate executable paths.
- Use decompilers and format-specific tools suitable for the artifact.

### 3. Hypothesis plan
For each question, record:
- Claim being tested.
- Evidence that would support it.
- Evidence that would weaken/falsify it.
- Smallest inspection needed.
- Stop condition.

### 4. Trace application-level behavior
- Inspect relevant methods and callers.
- Trace source → transformations → persistence/serialization → network/API sink → response/error handling.
- Record endpoint, method, headers, body fields and retry/error behavior only when directly supported.
- For collection/monitoring, distinguish event processing, local storage, upload initiation and confirmed successful transmission.

### 5. Inspect native code selectively
- Generate a demangled symbol inventory.
- Search for behavior-specific anchors.
- Disassemble targeted address ranges.
- Follow actual call paths and data movement.
- Generate full disassembly once per artifact/build and retain the artifact hash.
- Validate every extracted review file by checking its size, function headers and content.

### 6. Test network/TLS hypotheses carefully
Distinguish:
- VPN routing or packet forwarding.
- DNS filtering / IP firewalling.
- The application's own outbound HTTPS client.
- Actual TLS interception of another application's connection.
A strong static claim of HTTPS interception should demonstrate a connected path such as accepting/terminating a client TLS session, presenting/using an interception certificate/key, completing a server-side handshake, and accessing plaintext application data. TLS APIs, a CA bundle, or a VPN route alone are insufficient. If the path is not demonstrated, say “not established,” not “impossible.”

### 7. Evidence ledger
Use one row per material claim:

| ID | Claim | Exact artifact/function/address | Observation | Confidence | Limitations / counter-evidence |
|---|---|---|---|---|---|

Confidence labels:
- **Confirmed from static artifact:** directly visible code/configuration.
- **Strongly supported:** multiple linked observations support the conclusion.
- **Inferred:** plausible, but a link is missing.
- **Unresolved / not established:** insufficient evidence.

### 8. Stop rule
Stop broad searching when:
- Every primary question has a supported answer or a clearly recorded unresolved status.
- Further searches are repetitive and unlikely to change the result.
- A remaining question needs runtime, device, network, server-side or additional artifact evidence.
- Time is being spent cataloguing symbols without connecting them to a hypothesis.

Move unresolved questions into limitations and specify the smallest useful next test.

### 9. Deliverables
Produce:
1. Executive summary with direct answers.
2. Artifact identity, hash, scope and exclusions.
3. Tools and versions.
4. Method and reproducible commands.
5. Architecture/data-flow map.
6. Findings table with confidence and evidence references.
7. Alternative explanations and limitations.
8. Conclusions separated from hypotheses.
9. Optional next-step test plan.
10. Appendix with curated outputs, function addresses and key files.

## PowerShell starter patterns (Windows)

```powershell
$root = "C:\path\to\case"
$artifact = "$root\sample.apk"
$out = "$root\analysis"
New-Item -ItemType Directory -Force $out | Out-Null

Test-Path $artifact
Get-FileHash $artifact -Algorithm SHA256
```

Locate tools instead of assuming paths:

```powershell
Get-Command jadx, java -ErrorAction SilentlyContinue
Get-ChildItem "$env:LOCALAPPDATA\Android\Sdk\ndk" -Directory -ErrorAction SilentlyContinue
```

Example targeted search in a generated disassembly:

```powershell
Select-String -Path "$out\full_disassembly.txt" `
  -Pattern 'target_symbol|TLS|certificate|upload|firewall' -Context 3,3
```

Example native disassembly (replace paths and addresses with verified values):

```powershell
& "$bin\llvm-nm.exe" -n -C $native |
  Set-Content "$out\native_symbols.txt"

& "$bin\llvm-objdump.exe" -d --demangle `
  --start-address=0xSTART --stop-address=0xEND $native |
  Set-Content "$out\focused_function.asm"
```

Always verify `$native`, `$bin`, address ranges and output contents before relying on the result. The command patterns are templates, not universal commands for every file format or CLI version.

## Report language examples

Prefer:
- “The inspected method constructs a POST request to…”
- “The code path initiates an upload when…”
- “The native function extracts A/AAAA record IPs and updates firewall state…”
- “HTTPS interception was not established by the reviewed call paths.”

Avoid:
- “The app definitely uploads every event” without runtime evidence.
- “The TLS symbols prove HTTPS interception.”
- “No reference was found, therefore the behavior is impossible.”
- Treating a local status flag as proof of remote success.

## Case-study lesson

In the Bark for Chromebooks investigation, static evidence supported accessibility/message monitoring and upload paths, broad VPN routing, native TCP relay behavior, and DNS-driven firewall updates. A CA bundle was passed to the native HTTP client and TLS symbols existed, but a complete path demonstrating decryption of other apps' HTTPS traffic was not found. The correct conclusion was “not established,” with runtime validation left as a separate, authorized follow-up if required.
