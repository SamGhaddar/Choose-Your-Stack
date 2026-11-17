# Choose Your Stack: instructions for Claude

This repo is Sam's personal reference for choosing tech stacks.

## When asked to pick or review a stack

1. Read `guides/reference-stacks.md` first, then the guide for each layer the project needs: `api-styles.md`, `databases.md`, `frontend-and-mobile.md`, `backend.md`, `hosting-and-deploy.md`, `auth-and-payments.md`. Use `glossary.md` for unfamiliar terms.
2. Ask only for missing facts that change the answer: clients (web/mobile/devices), real-time needs, third-party integrations, target country (payments and hosting availability), team size, existing stack.
3. Recommend one main choice per layer plus only the add-ons the project needs. Explain each choice using the guides' criteria and name the closest row in the Reference stacks table.
4. Flag the refactoring symptoms and the regrets most likely to show up later.
5. Offer to record the result as `decisions/<project>.md` using `decisions/TEMPLATE.md`, and add a row to `decisions/README.md`.

## When editing guides

- Keep the structure: quick picker, comparison table, each option in short, reference stacks, refactoring signals, myths.
- Prefer tables and diagrams over long prose. Sam is a visual learner.
- Keep guides generic: do not use Sam's own projects as examples.
- Update the `Last updated` date at the top of the guide you changed.
