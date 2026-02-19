# Contributing to Rokid Docs

Thank you for your interest in contributing to Rokid Docs! This project exists because Rokid development documentation is scattered and hard to find. Every contribution helps the community build better apps for Rokid AR glasses.

## How to Contribute

### Reporting Issues

- **Incorrect documentation**: Open an issue if you find errors, outdated information, or misleading instructions
- **Missing documentation**: Request docs for SDK features, hardware capabilities, or workflows that aren't covered yet
- **Unclear explanations**: If something is confusing, it's a bug in the docs

### Submitting Changes

1. **Fork** the repository
2. **Create a branch** from `main` (`git checkout -b docs/your-topic`)
3. **Make your changes** following the style guidelines below
4. **Submit a pull request** with a clear description of what you changed and why

### What We Need Help With

- **English translations** of Chinese-language SDK documentation
- **Getting started guides** and tutorials for common development tasks
- **Code examples** demonstrating SDK usage (CXR-M, CXR-S, CXR-L)
- **Troubleshooting guides** based on your development experience
- **API documentation** for undocumented or partially documented features
- **Hardware findings** from your own reverse engineering or development
- **Corrections** to any inaccurate information in existing docs

## Style Guidelines

### Markdown

- Use ATX-style headers (`#`, `##`, `###`)
- Use fenced code blocks with language identifiers (````java`, ````kotlin`, ````xml`)
- Use tables for structured data (specs, API parameters, etc.)
- Use relative links between documentation files
- Keep line lengths reasonable (no hard wrap requirement, but break at logical points)

### File Organization

- **SDK documentation** goes in the appropriate SDK directory (`cxr-m/`, `cxr-s/`, `cxr-l/`)
- **YodaOS system docs** go in `yodaos/docs/` under the appropriate subdirectory
- **New top-level sections** should be discussed in an issue first
- File names use lowercase with hyphens: `my-new-doc.md`

### Content

- Be specific and technical - this is developer documentation
- Include code examples where possible
- Note which SDK version or firmware version information applies to
- If information was obtained through reverse engineering, note that clearly
- Link to related docs within the repo

## Decompiled Sources

The `decompiled/` directories and `yodaos/DECOMPILED*/` contain reverse-engineered source code tracked with Git LFS. If you're adding findings from decompiled sources:

1. Document your findings in the relevant markdown file
2. Reference specific classes/methods by their full package path
3. Note the firmware version the decompilation was performed on

## Code of Conduct

This project follows the [Contributor Covenant Code of Conduct](CODE_OF_CONDUCT.md). By participating, you agree to uphold this code.

## Questions?

Open a GitHub Discussion or Issue if you have questions about contributing. There are no bad questions - if you're confused about something, other developers probably are too.
