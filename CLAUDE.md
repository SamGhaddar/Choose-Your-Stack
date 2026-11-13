# Choose Your Stack: instructions for Claude

This repo is Sam's personal reference for choosing tech stacks.

## When asked to pick or review a stack

1. Read the relevant guide in `guides/` (start with `guides/api-styles.md` for anything with an API).
2. Ask only for missing facts that change the answer: clients (web/mobile/devices), real-time needs, third-party integrations, team size, existing stack.
3. Recommend one main choice per layer plus only the add-ons the project needs. Explain each choice using the guide's criteria and name the closest row in its Reference stacks table.
4. Flag the refactoring symptoms most likely to show up later.
5. Offer to record the result as `decisions/<project>.md` using `decisions/TEMPLATE.md`, and add a row to `decisions/README.md`.

## When editing guides

- Keep the structure: quick picker, comparison table, each option in short, reference stacks, refactoring signals, myths.
- Prefer tables and diagrams over long prose. Sam is a visual learner.
- Update the `Last updated` date at the top of the guide you changed.
