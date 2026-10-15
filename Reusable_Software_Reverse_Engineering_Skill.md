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


---

## Appendix A — Cross-Framework Mobile Application Analysis (added 2026-10-15)

### A1. Identify the application architecture before selecting tools

Classify the artifact before doing broad disassembly. Look for:
- Android manifest package/version, activities, services, receivers, providers, permissions and exported components.
- Runtime/framework markers: React Native, Expo, Hermes, Flutter/Dart, Xamarin/.NET, native Kotlin/Java, Unity, or a hybrid combination.
- Bundle/bytecode assets, native libraries, split APKs, dynamic feature modules and remote configuration.
- Build type, architecture flags and version metadata.

Do not assume that the presence of a framework/library means every associated capability is active. Distinguish packaged, registered, reachable and observed behavior.

### A2. React Native / Expo / Hermes workflow

For React Native applications:
1. Identify the JavaScript bundle and Hermes bytecode version. Record file size, hash, extraction path, tool versions and warnings.
2. Try supported source-map or bundle inspection options first; use a Hermes-aware decompiler only when needed.
3. Preserve the original bundle. Treat decompiled output as a lossy representation, especially when the bytecode version is newer than the decompiler or unsupported instructions/registers are reported.
4. Search targeted anchors (operation names, endpoint keys, native module names, event names, storage keys), then inspect surrounding function/control-flow context.
5. Validate critical claims against native Java/Kotlin/smali or independent strings/config evidence. A string match alone is not a call path.
6. If the decompiler produces malformed or incomplete code, report the limitation and use bytecode/smali/string-level corroboration rather than treating reconstructed JavaScript as source truth.

For Expo applications, inspect both Expo module registration and any custom React Native `ReactPackage` registration. They are distinct registration paths. Check the Android manifest and `MainApplication`/smali for package registration when JADX source is missing or decompilation is incomplete.

### A3. Native-module bridge trace

For each custom module, build a trace with these columns:

| Stage | Required evidence |
|---|---|
| Definition | Native module name, exported methods and argument/result signatures |
| Registration | Package/module registration in application bootstrap |
| JavaScript use | Actual bridge lookup and call site; account for minification/decompiler transformations |
| Native guard | Permission, authentication, feature-flag, lifecycle or argument checks |
| Side effect | Storage, worker scheduling, system API, network request or notification |
| Completion | Promise resolution/rejection, callback, event or persisted status |
| Confidence | Confirmed, strongly supported, inferred or unresolved |

Registration proves availability to the bridge, not that the JavaScript application invokes the module. A native method definition proves capability, not runtime execution.

### A4. Runtime configuration and endpoint provenance

Track endpoint values separately:
- hardcoded/build-time default;
- remote configuration URL and fetched payload;
- local persisted override/cache;
- runtime-selected value;
- endpoint constructed by a helper;
- endpoint passed to the particular request;
- endpoint observed during authorized runtime testing, if performed.

Record fetch timeout, cache behavior, fallback behavior, URL normalization and scheme conversion when supported. Do not describe a remotely fetched configuration snapshot as permanent or necessarily current. Do not equate a configured base URL with a verified request URL unless the call path is traced.

### A5. Authentication and local secret storage

Trace token acquisition, storage key, storage API, options, retrieval, refresh/rotation, request attachment, retry behavior, logout cleanup and failure handling. Verify native implementation and options when the security claim matters. For example, the existence of an AES-GCM encryptor in a secure-storage library does not prove that a particular item used that encryptor or required biometric authentication.

Separate:
- access token vs refresh token;
- token present in storage vs session authenticated;
- refresh attempt vs successful refresh;
- secure-storage library capability vs actual configuration for this key;
- a request retry vs confirmed authorization success.

### A6. Personal-data and background synchronization trace

For contact, location, media, clipboard or other personal-data paths, trace:
1. source API and required permission;
2. selection/filtering and data fields;
3. transformations, normalization, hashing or redaction;
4. local persistence and retention/deletion behavior;
5. batch size and request serialization;
6. destination URL, method, headers and body;
7. trigger/scheduler and retry/backoff conditions;
8. response parsing, per-record results and local state updates.

Report the minimum proven claim. Contact collection code plus an upload client establishes an implemented path; it does not prove that a real device ran it, that a server received it, or how the server retains the data.

### A7. Realtime and GraphQL architecture

Inventory GraphQL operations by feature and distinguish HTTP GraphQL requests from WebSocket/realtime messages. For WebSockets, record URL construction, authentication/initialization protocol, channel names, event types, reconnect/heartbeat logic and replay/resync mechanisms only where supported. Event names and operation strings alone do not establish a successful connection or server-side behavior.

### A8. Evidence-backed architecture diagrams

Build a component inventory and diagram after the major call paths are understood. Use:
- solid connectors for directly supported relationships;
- dashed connectors for inferred/conditional relationships;
- labels for protocol or data type (e.g., GraphQL/HTTPS, WebSocket, FCM, local secure storage);
- a legend and a statement that the diagram represents static architecture, not an observed live trace;
- separate client, native platform, remote configuration, backend and third-party service boundaries.

Every material arrow should map to one or more entries in the evidence ledger. Omit or label uncertain endpoints rather than inventing backend internals.

### A9. Report production and closure gate

Before finalizing:
- verify artifact hash and exact product/version;
- record analysis date and tools/versions, including failed/partial decompiler steps;
- include a concise executive summary, scope, architecture diagram, findings/evidence table, limitations, and smallest useful follow-up tests;
- distinguish static findings, externally fetched configuration snapshots, runtime observations and analyst inference;
- validate all report tables, diagram legibility, file paths, links and generated document opening;
- stop broad exploration when every primary hypothesis has a supported answer or explicit unresolved status and further searches are unlikely to change the conclusion.

### A10. Matiks case-study lessons (generalizable)

The Matiks 1.49.0 analysis reinforced these practices:
- a Hermes bundle can be very large and decompilation may be incomplete; use targeted searches and native-code corroboration;
- custom React Native packages may be registered in smali even when the application bootstrap source is absent from JADX;
- remote client configuration can override hardcoded defaults, so report both provenance and timestamp;
- contact synchronization should be traced from Android provider through local pending storage, GraphQL serialization, authorization, response handling and retries;
- static code establishes implemented capability, not successful runtime execution or backend retention.
