# docs-researcher

External documentation lookup — find and present official docs for libraries, packages, tools, and APIs from the internet.

## Components

- **documentation-researcher-agent** — Autonomous agent that finds and presents official documentation from the internet
- **web-docs-search** — Skill with search strategies and source hierarchies for finding external library documentation

## Usage

The documentation-researcher agent activates automatically when you:

- Ask how to use an external library method (e.g., "How do I use lodash debounce?")
- Encounter an error from an external dependency and ask for help
- Request API reference or docs for a package

The agent searches official documentation sources, fetches relevant sections, and returns structured results with signatures, parameters, examples, and source URLs.
