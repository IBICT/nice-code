<div align="center">

<h1 align="center">Nice Code</h1>

<div align="center">

<a target="_blank" href="https://opensource.org/licenses/Apache-2.0" style="background:none">
    <img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" style="height: 22px;" />
</a>
<a target="_blank" href="https://nice.ibict.br/changelog" style="background:none">
    <img src="https://img.shields.io/badge/changelog-%2396EFF3" style="height: 22px;" />
</a>

<p></p>

**Source-controlled AI checks, enforceable in CI**

</div>

![Banner](https://oasisbr.ibict.br/vufind/themes/oasisbr/images/logoibictversao1.png?_=1761139737)

## Getting started

Paste this into your coding agent of choice:

```
Help me write checks for this codebase: https://nice.ibict.br
```

## How it works

Nice Code runs agents on every pull request as GitHub status checks. Each agent is a markdown file. Green if the code looks good, red with a suggested diff if not. Here is an example that performs a security review:

```yaml
---
name: Security Review
description: Review PR for basic security vulnerabilities
---
Review this PR and check that:
  - No secrets or API keys are hardcoded
  - All new API endpoints have input validation
  - Error responses use the standard error format
```

## VS Code Agent

Work on development tasks together with AI directly from VS Code.

## VS Code Chat

Ask general questions and clarify code sections without leaving your editor.

## VS Code Edit

Modify a code section inline without leaving your current file.

## VS Code Autocomplete

Receive intelligent inline code suggestions as you type.

## Contributing

Read the [contributing guide](https://github.com/ibict/nice-code/blob/main/CONTRIBUTING.md) and
join the [GitHub Discussions](https://github.com/ibict/nice-code/discussions).

## License

[Apache 2.0 © 2026 IBICT.](./LICENSE.txt)
