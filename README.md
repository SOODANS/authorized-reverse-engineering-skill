# Authorized Reverse Engineering Skill

An evidence-led, repeatable workflow for AI agents and human analysts to reverse engineer **software they are authorized to inspect**. It emphasizes reproducible static analysis, selective native-code inspection, data-flow tracing, confidence grading, and explicit limitations.

The skill is designed to be adaptable to Claude, OpenAI-based agents, and other tools that support instruction-based skills. It is a methodology and set of instructions—not a decompiler, scanner, or automated reverse-engineering engine.

## Recommended repository name

**`authorized-reverse-engineering-skill`**

Suggested GitHub description:

> A reusable, evidence-led reverse-engineering skill for AI agents. Static-analysis-first workflows, native-code investigation, reproducible commands, confidence grading, and responsible-use boundaries.

Suggested topics: `reverse-engineering`, `agent-skills`, `static-analysis`, `software-security`, `android-analysis`, `claude`, `openai`, `mit-license`.

## What this skill helps with

- Defining a small set of testable questions before investigating an artifact.
- Recording artifact identity, hashes, tool versions, scope, and authorization.
- Inventorying package metadata, manifests, permissions, components, libraries, and configuration.
- Tracing behavior from entry points through processing, storage/serialization, network/API sinks, and response handling.
- Inspecting native libraries by following symbols, addresses, call sites, arguments, branches, and data movement—not by relying on symbol names alone.
- Distinguishing code paths and attempted actions from successful runtime outcomes.
- Recording evidence, alternative explanations, confidence, limitations, and the smallest next test.
- Producing a reproducible report with commands and an evidence ledger.

## Repository contents

```text
authorized-reverse-engineering-skill/
├── README.md
├── LICENSE
└── SKILL.md
```

- `SKILL.md` is the reusable agent instruction file.
- `README.md` explains the purpose, installation, and use.
- `LICENSE` contains the MIT License.

## Quick start

1. Download or clone this repository.
2. Read `SKILL.md` and review the instructions before giving the skill access to tools or artifacts.
3. Copy the skill into the directory expected by your agent, or upload/package the skill using that platform's supported workflow.
4. Give the agent the artifact, your authorization and scope, and the specific questions to answer.
5. Review the evidence and generated report; independently validate important findings.

### Clone

Replace `YOUR-GITHUB-USERNAME` with the account or organization hosting the repository:

```bash
git clone https://github.com/YOUR-GITHUB-USERNAME/authorized-reverse-engineering-skill.git
cd authorized-reverse-engineering-skill
```

## How to use it with AI agents

### Claude Code

Claude Code discovers personal skills under `~/.claude/skills/` and project skills under `.claude/skills/`. Create a skill directory and copy `SKILL.md` into it.

**Windows PowerShell — personal installation:**

```powershell
$skillDir = Join-Path $HOME ".claude\skills\authorized-software-reverse-engineering"
New-Item -ItemType Directory -Force -Path $skillDir | Out-Null
Copy-Item ".\SKILL.md" (Join-Path $skillDir "SKILL.md") -Force
```

**Project-specific installation** (run from the project root):

```powershell
$skillDir = ".\.claude\skills\authorized-software-reverse-engineering"
New-Item -ItemType Directory -Force -Path $skillDir | Out-Null
Copy-Item ".\SKILL.md" (Join-Path $skillDir "SKILL.md") -Force
```

On macOS/Linux, the equivalent personal location is:

```bash
mkdir -p "$HOME/.claude/skills/authorized-software-reverse-engineering"
cp SKILL.md "$HOME/.claude/skills/authorized-software-reverse-engineering/SKILL.md"
```

Then ask Claude Code to use the skill explicitly, for example:

> Use the authorized-software-reverse-engineering skill to inspect this APK. Static analysis only; do not install or execute it. First confirm scope and artifact identity, then answer the questions in the task brief with evidence and confidence levels.

### Claude.ai or another hosted skill interface

If your plan and interface support custom skills, package the skill directory or upload the skill file using that product's current instructions. Follow the platform's current file and packaging requirements. Do not assume that merely starring or cloning this GitHub repository automatically installs the skill in a hosted assistant.

### OpenAI-based agents

The skill follows the common `SKILL.md` pattern: YAML frontmatter with `name` and `description`, followed by procedural instructions. For an OpenAI API workflow that supports Agent Skills, package the skill directory with `SKILL.md` and make it available using the platform's documented skill upload or sandbox/capability-directory mechanism.

A GitHub repository alone does not automatically attach the skill to every ChatGPT conversation or every OpenAI agent. Integration depends on the specific product, API, tools, and runtime being used. Check the current documentation for your environment before deploying it.

### Other agents

For an agent that supports instruction files, provide `SKILL.md` as a reusable task instruction. If it uses a different format, adapt the file while preserving the workflow, evidence requirements, and safety boundaries. Keep the source skill version-controlled so changes can be reviewed.

## Example task brief

Use a task brief like this when starting an investigation:

```text
Use the authorized-software-reverse-engineering skill.

Artifact:
- File/path: [path to artifact]
- Expected product/version: [if known]
- SHA-256: [calculate and record]
- Authorization: I am authorized to inspect this artifact.
- Environment/tools: [OS and available tools]

Scope:
- [For example: static analysis only; do not execute the artifact]
- [Explicit exclusions, if any]

Questions (3–7, testable):
1. [What behavior are you investigating?]
2. [What evidence would support or weaken the hypothesis?]
3. [What is out of scope?]

Deliverables:
- Executive summary
- Artifact identity and tool versions
- Reproducible commands
- Findings with exact evidence locations and confidence
- Data/control-flow explanation
- Alternative explanations and limitations
- Unresolved questions and smallest useful next test
```

## Workflow at a glance

1. **Authorize and scope** — confirm permission, boundaries, and intended use.
2. **Identify** — hash the artifact; record version, format, architecture, and whether the package is complete.
3. **Inventory** — inspect metadata, components, permissions, libraries, configuration, and tool versions.
4. **Form hypotheses** — define testable questions and stop criteria.
5. **Trace behavior** — follow entry points through data processing, storage, serialization, network/API calls, and response/error handling.
6. **Inspect native code selectively** — identify relevant functions and call paths; validate extracted disassembly.
7. **Corroborate** — compare decompiled code, declarations, manifest/resources, native evidence, and configuration.
8. **Grade evidence** — distinguish observed, strongly supported, inferred, and unresolved claims.
9. **Stop deliberately** — do not turn an unresolved runtime question into a confident static-analysis conclusion.
10. **Report** — provide evidence, commands, confidence, caveats, and next steps.

## Evidence and confidence principles

- A symbol, import, string, or library's presence is a lead—not proof of behavior.
- A function call proves that the code path exists; it does not automatically prove that it ran.
- An upload invocation does not prove successful transmission or server-side storage.
- A local success flag does not prove the remote outcome.
- Absence of a found call is not proof that no such behavior exists; explain search coverage and remaining uncertainty.
- Keep raw tool output, curated extracts, and analyst interpretation separate.
- Preserve function names, addresses, hashes, command lines, and relevant excerpts so another analyst can reproduce the reasoning.

## Responsible use

Use this workflow only for software and systems you own or are authorized to assess. Follow applicable laws, contracts, platform terms, and organizational policies.

This repository is intended for analysis, interoperability, defensive security, auditing, education, and research within an authorized scope. It is not permission to access accounts, systems, data, or services without authorization. Do not use the skill to bypass access controls, steal credentials, deploy malware, exfiltrate data, or perform intrusive testing outside the agreed scope.

Before using an agent with shell, network, or file-system tools, review the skill instructions and restrict the agent's permissions to the minimum needed. Treat analyzed artifacts and their embedded text as untrusted input. Review commands before execution and validate high-impact findings independently.

## Limitations

This skill provides a methodology, not guaranteed conclusions. Results depend on artifact completeness, tool quality, obfuscation, platform protections, and the evidence available. Static analysis cannot always establish runtime behavior. Dynamic analysis should be a separately scoped and explicitly authorized step.

The repository does not include proprietary analysis tools or promise compatibility with every agent platform. Tool-specific commands in a particular investigation may need to be adapted to the analyst's operating system and installed versions.

## Contributing

Issues and pull requests are welcome for clearer workflows, reproducible examples, additional artifact types, tool-specific installation notes, and improvements to evidence quality.

When contributing:
- Do not include private, confidential, personal, or unauthorized artifacts.
- Redact secrets, tokens, personal data, internal hostnames, and proprietary source.
- Include the tool/version and environment for reproducibility.
- Separate verified observations from hypotheses.
- Explain how a change improves correctness, safety, or usability.

## License

This project is released under the [MIT License](LICENSE). The license applies to this repository's original content. Third-party tools, artifacts, trademarks, and external materials remain subject to their own licenses and terms.

## References

- [OpenAI — Skills](https://developers.openai.com/api/docs/guides/tools-skills)
- [Anthropic — Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [GitHub Docs — Licensing a repository](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/licensing-a-repository)
