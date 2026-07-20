---
name: implementation-quality-pass
description: Use after writing or modifying non-trivial application code, before final verification, to find and fix common AI-generated code problems.
---

# Implementation Quality Pass

Inspect the full implementation diff and relevant surrounding code. Apply each question below and fix concrete issues before final verification.

Keep fixes scoped to the implementation. These questions are prompts for judgment, not mechanical lint rules.

## Is This Code Properly Organized?

Organization should communicate what the code means, what owns it, and how it relates to the rest of the repository.

Ask:

- Does this file or code block represent a meaningful concept, or does it exist here only because nearby code uses it?
- Does the code fit the responsibility of its containing file or module?
- Does established comparable code use a different naming, placement, or organizational pattern without a clear reason? For example, equivalent schemas may be named or exported differently, or equivalent features may use inconsistent file boundaries.
- Can an extracted file, function, type, or code block be named after a coherent concept, or only after its implementation details or current caller?
- Does the resulting import or dependency direction make the ownership of the code unclear or surprising?

Improve it:

- Move the code to the module that owns the represented concept, rule, or source of data.
- Match the naming, placement, and organization of established comparable code unless the new code has a concrete reason to differ.
- If the code has no clear semantic owner, reconsider the abstraction or API instead of placing it in a generic utility or helper location.
- Keep code local when it has one natural owner and no independent meaning, provided it remains readable, does not pollute the containing file, and established patterns do not place this kind of code elsewhere.
- Extract code when the extracted unit can be named clearly and has a coherent responsibility, keeps its containing file focused, or follows established precedent for that kind of code.
- Recheck imports and exports after reorganizing the code so its public API and dependency direction remain intentional.

## Should This Code Derive From an Existing Source of Truth?

When multiple representations express the same data or rule, determine whether one should be authoritative and the others derived so changes propagate automatically and cannot become stale.

Ask:

- Does the new code redefine a value, type, schema, mapping, or rule that already exists elsewhere?
- Is the same data or domain decision represented independently in multiple places? If so, should one representation be authoritative and the others derive from it?
- Are runtime and static representations maintained separately when one could be derived from the other? For example, is a union type restating the values already defined by a schema or constant collection?
- Are derived values stored, passed, or returned alongside the data needed to calculate them?
- Would changing one concept require coordinated edits in multiple locations?
- If two representations disagree, is it unclear which one is authoritative?

Improve it:

- Identify the representation closest to the concept's semantic owner or external authority and make it canonical.
- Derive types, schemas, constants, collections, and mappings from the canonical representation using established repository patterns.
- Move shared domain decisions to the module that owns the rule and have callers consume that decision.
- Remove redundant fields, parameters, and stored state when the value can be derived safely and clearly from authoritative data.
- Keep conversions at system boundaries rather than allowing parallel representations to spread through the program.
- If derivation requires a disproportionately large or indirect transformation, keep the representations separate and reconsider whether they truly share an authority; that complexity suggests the assumed source-of-truth relationship may be wrong.

## Is This Code Making Effective Use of Existing Libraries and Abstractions?

Established libraries and project abstractions should be used instead of custom implementations of behavior they already represent.

Ask:

- Does the code manually implement behavior already provided by a project dependency or shared abstraction?
- Does imperative control flow reproduce an established declarative operation for validation, matching, optional values, errors, resources, retries, or concurrency? A common example is manually orchestrating behavior that a library module or combinator already represents directly, such as an `Effect` module when Effect is in use.
- Does the code represent or compose a concern ad hoc when the repository has an established model for it? Errors, state, services, schemas, and effects are common examples, but not an exhaustive list.
- Does comparable established code use a common library or abstraction while the new code invents a different approach?

Improve it:

- Replace custom behavior with the established library API or project abstraction.
- Use dependencies according to established repository patterns and their intended level of abstraction.
- Express validation, errors, state transitions, control flow, and similar concerns through the project's existing models and library APIs.
- Prefer an already-adopted project abstraction over introducing another dependency for the same capability.
- Use a custom implementation only when the available API does not support the use case or is such a poor fit that using it makes the behavior less clear.

## Does This Code Model a Meaningful Concept?

Functions, types, and APIs should communicate concepts and operations that matter to the program, not merely connect one piece of implementation to another.

Ask:

- Does a generic verb such as `resolve`, `process`, `handle`, `build`, or `transform` hide the actual decision or operation being performed?
- Do types such as `XInput` and `XOutput` group values for implementation convenience rather than representing a readable function contract, meaningful request, result, state, or boundary?
- Does a function accept or return a newly invented shape when an existing domain value, schema, or service contract already represents the concept?
- Does a helper exist only to make a sequence of implementation steps callable from elsewhere, without giving that sequence clearer meaning?
- Would a caller need to read the implementation to understand what the function guarantees or why its inputs belong together?

Improve it:

- Identify the domain action, rule, capability, or boundary represented by the code and name the operation after it.
- Use existing domain types and authoritative values instead of unpacking them into arbitrary fields and repackaging them into new containers.
- Replace generic input and output bags with meaningful request, result, state, or boundary types when those concepts genuinely exist.
- Inline or remove glue abstractions that add no meaningful contract and make the data flow harder to follow.
- When a transformation crosses a genuine boundary, model each side according to the concept it represents and make the conversion explicit.
- Keep mechanical helpers when they materially improve readability, but name them after the exact operation and avoid promoting incidental implementation shapes into program concepts.

## Is This Control Flow Clear and Declarative?

Treat control flow as suspect when ternaries, boolean combinations, or imperative branching make meaningful cases difficult to see, or when repository-, platform-, or library-native abstractions could represent them directly.

When the code is unclear:

- Prefer established declarative abstractions, such as applicable library modules and combinators, including `Effect` modules when Effect is in use.
- Otherwise, make the cases explicit with simple, readable branching or meaningful state types.
- Keep direct conditionals and ternaries when they remain the clearest representation.

## Is This Code More Complicated Than It Needs to Be?

Prefer the simplest code that clearly solves the current problem.

Ask:

- Does the code add fallbacks, normalization, or validation for states already ruled out by an authoritative boundary?
- Does it preserve an old API or behavior without a concrete requirement to do so?
- Do intermediate transformations or nearly identical branches obscure a more direct implementation?
- Does a single-use abstraction make its caller harder rather than easier to understand?

Improve it:

- Remove speculative flexibility, compatibility, and defensive behavior that serves no concrete requirement.
- Rely on guarantees already enforced by schemas, types, or system boundaries.
- Preserve complexity that belongs to the problem; do not replace clear structure with dense or clever code.

## Finish

Re-read the resulting diff, fix any remaining issues, then run the repository's required verification.
