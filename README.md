# claude-skills

A collection of agent skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that extend its capabilities across planning, design, development, and refactoring.

Inspired by Matt Pocock's [skills](https://github.com/mattpocock/skills) repository.

## Skills

### design-an-interface

Generate multiple radically different interface designs for a module using parallel sub-agents. Implements the "Design It Twice" methodology from *A Philosophy of Software Design*, then compares designs on simplicity, flexibility, efficiency, and ease of use.

### grill-me

Interview you relentlessly about a plan or design until reaching shared understanding. Walks down each branch of the decision tree one-by-one, stress-testing assumptions and surfacing gaps.

### tdd

Test-driven development with a strict red-green-refactor loop. Enforces vertical slices (one test, one implementation, repeat) over horizontal slices. Includes reference material on testing patterns, mocking boundaries, interface design, and refactoring.

### refactor

Identify refactor opportunities across a codebase following language-specific best practices. Auto-detects the project language and runs through four phases: discover, discuss, plan, and execute. Ships with configs for Rust, TypeScript, Vue, and Tailwind CSS.

### write-a-skill

Create new agent skills with proper structure, progressive disclosure, and bundled resources. Guides you through requirements gathering, drafting, and review to produce a well-structured skill directory.

## Installation

Add individual skills to your Claude Code configuration:

```sh
npx skills@latest add OzCroot/claude-skills/<skill-name>
```

For example:

```sh
npx skills@latest add OzCroot/claude-skills/tdd
```

## License

MIT
