# Claude Rules for This Project

## Workflow — Mandatory

- **NEVER implement anything** without first stating a plan and receiving explicit approval from the user.
- **NEVER write or modify code** unless the user has said yes to the proposed plan.
- If a request is ambiguous, **ask a clarifying question** before doing anything.
- If there are multiple valid approaches, **list them and wait** for the user to choose one.

## Scope — Mobile vs Desktop

- **Mobile-only changes go exclusively inside `@media (max-width: 600px)`.**
- **NEVER modify global CSS or shared HTML structure** for a mobile-only request.
- If a mobile change REQUIRES touching shared HTML (affects both views), **STOP — explain why and ask the user how to proceed.**
- Desktop layout is never touched unless the user explicitly says so.

## Keywords

- **"Plan only"** — Respond with a plan. Do not write any code.
- **"What's your approach?"** — Same as above. Describe the approach only.
- **"Confirm"** or **"Go ahead"** — Permission to implement the previously stated plan.

## Project Reference

- See `PROJECT.md` for architecture, conventions, and feature status.
- Single-file app: all HTML, CSS, JS in `index.html`. No build step.
- All mobile responsive changes: `@media (max-width: 600px)` block in `index.html`.
