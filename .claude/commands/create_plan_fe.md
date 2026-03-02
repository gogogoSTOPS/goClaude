---
description: Create detailed implementation plans for Flutter frontend features with thorough research and iteration
model: opus
---

# Frontend Implementation Plan

You are tasked with creating detailed implementation plans for the **goCREW Flutter app** — which uses **GetX** for state management/DI/routing and **Clean Architecture** (domain/data/presentation layers) with functional error handling via `dartz` (`Either<Failure, Success>`).

You should be skeptical, thorough, and work collaboratively with the user to produce high-quality technical specifications that respect the app's architecture.

## Initial Response

When this command is invoked:

1. **Check if parameters were provided**:
   - If a file path or ticket reference was provided as a parameter, skip the default message
   - Immediately read any provided files FULLY
   - Begin the research process

2. **If no parameters provided**, respond with:
```
I'll help you create a detailed frontend implementation plan for goCREW. Let me start by understanding what we're building.

Please provide:
1. The task/ticket description (or reference to a ticket file)
2. Any relevant context, constraints, or specific requirements
3. Links to related research, Figma designs, or previous implementations

I'll analyze this information and work with you to create a comprehensive plan.

Tip: You can also invoke this command with a ticket file directly: `/create_plan_fe thoughts/allison/tickets/eng_1234.md`
For deeper analysis, try: `/create_plan_fe think deeply about thoughts/allison/tickets/eng_1234.md`
```

Then wait for the user's input.

## Project Architecture Reference

```
lib/
├── base/           # Base classes, constants, DI bindings
│   ├── bindings/   # GetX DI (AppBindings orchestrates all sub-bindings)
│   ├── constants/  # Colors, sizes, styles, images, texts
│   └── use_case/   # BaseUseCase<Type, Params>, Failure hierarchy
├── data/           # Data layer
│   ├── data_sources/
│   │   ├── remote/   # API calls (abstract + impl per feature)
│   │   └── local/    # SQLite via DAO pattern
│   ├── models/      # Extend domain entities, add fromJson/toJson
│   ├── network/     # Dio setup, ApiClient, interceptors, API endpoints
│   └── repositories/ # Implement domain repository interfaces
├── domain/         # Domain layer (PURE — no Flutter/GetX/Dio imports)
│   ├── entities/    # Plain Dart classes (no JSON)
│   ├── repositories/ # Abstract repository interfaces
│   └── use_case/    # UseCase<ReturnType, Params> → Either<Failure, T>
├── view/           # UI widgets (GetView<Controller>, Obx)
├── view_model/     # GetxControllers (depend on use cases only)
├── routes/         # GetPage definitions, route names, arguments
├── services/       # App-level services (storage, user, notifications, firebase)
├── analytics/      # Mixpanel, MS Clarity
├── utils/          # Extensions, enums, helpers
├── values/         # Environment config (env.dart)
└── widgets/        # Reusable widgets
```

**DI Binding Order:** Core services → Network (Dio/ApiClient) → DataSources → Repositories → UseCases → Services → Controllers

**Key patterns:**
- `Either<Failure, T>` for all repository/use case returns (dartz)
- Failure types: `ApiFailure`, `InternetFailure`, `LocalDBFailure`
- Two Dio instances: authenticated (`AuthInterceptor`) and public
- Tagged ApiClients: `Get.find<ApiClient>(tag: 'auth')` vs `tag: 'public'`
- GetStorage for key-value, SQLite (sqflite) for structured data
- `flutter_screenutil` for responsive sizing

## Process Steps

### Step 1: Context Gathering & Initial Analysis

1. **Read all mentioned files immediately and FULLY**:
   - Ticket files (e.g., `thoughts/allison/tickets/eng_1234.md`)
   - Research documents, Figma specs
   - Related implementation plans
   - Any JSON/data files mentioned
   - **IMPORTANT**: Use the Read tool WITHOUT limit/offset parameters to read entire files
   - **CRITICAL**: DO NOT spawn sub-tasks before reading these files yourself in the main context
   - **NEVER** read files partially - if a file is mentioned, read it completely

2. **Spawn initial research tasks to gather context**:
   Before asking the user any questions, use specialized agents to research in parallel:

   - Use the **codebase-locator** agent to find all files related to the ticket/task
   - Use the **codebase-analyzer** agent to understand how the current implementation works
   - Use the **codebase-pattern-finder** agent to find similar existing screens/features to model after
   - If relevant, use the **thoughts-locator** agent to find any existing thoughts documents about this feature

   These agents will:
   - Find relevant source files, configs, and tests
   - Trace data flow and key functions
   - Return detailed explanations with file:line references
   - **Find existing reusable widgets, themes, and components** (critical — see Reuse-First Principle below)

3. **Read all files identified by research tasks**:
   - After research tasks complete, read ALL files they identified as relevant
   - Read them FULLY into the main context
   - This ensures you have complete understanding before proceeding

4. **Analyze and verify understanding**:
   - Cross-reference the ticket requirements with actual code
   - Identify any discrepancies or misunderstandings
   - Note assumptions that need verification
   - Determine true scope based on codebase reality
   - **Catalog existing widgets and themes that can be reused** (see Reuse-First Principle)

5. **Present informed understanding and focused questions**:
   ```
   Based on the ticket and my research of the codebase, I understand we need to [accurate summary].

   I've found that:
   - [Current implementation detail with file:line reference]
   - [Relevant pattern or constraint discovered]
   - [Existing reusable widgets/themes that apply]

   Questions that my research couldn't answer:
   - [Specific technical question that requires human judgment]
   - [Business logic clarification]
   - [Design preference that affects implementation]
   ```

   Only ask questions that you genuinely cannot answer through code investigation.

### Step 2: Research & Discovery

After getting initial clarifications:

1. **If the user corrects any misunderstanding**:
   - DO NOT just accept the correction
   - Spawn new research tasks to verify the correct information
   - Read the specific files/directories they mention
   - Only proceed once you've verified the facts yourself

2. **Create a research todo list** using TodoWrite to track exploration tasks

3. **Spawn parallel sub-tasks for comprehensive research**:
   - Create multiple Task agents to research different aspects concurrently
   - Use the right agent for each type of research:

   **For deeper investigation:**
   - **codebase-locator** - To find more specific files (e.g., "find all files that handle [specific component]")
   - **codebase-analyzer** - To understand implementation details (e.g., "analyze how [system] works")
   - **codebase-pattern-finder** - To find similar features we can model after

   **For reuse discovery (MANDATORY):**
   - **codebase-pattern-finder** - Search `lib/widgets/` for reusable widgets that match the UI needs
   - **codebase-pattern-finder** - Search `lib/base/constants/` for existing themes, colors, text styles, sizes
   - **codebase-analyzer** - Analyze how similar screens are built to follow the same patterns

   **For historical context:**
   - **thoughts-locator** - To find any research, plans, or decisions about this area
   - **thoughts-analyzer** - To extract key insights from the most relevant documents

   Each agent knows how to:
   - Find the right files and code patterns
   - Identify conventions and patterns to follow
   - Look for integration points and dependencies
   - Return specific file:line references
   - Find tests and examples

3. **Wait for ALL sub-tasks to complete** before proceeding

4. **Present findings and design options**:
   ```
   Based on my research, here's what I found:

   **Current State:**
   - [Key discovery about existing code]
   - [Pattern or convention to follow]

   **Reusable Components Found:**
   - [Widget from lib/widgets/ that can be reused]
   - [Theme/style from lib/base/constants/ that applies]
   - [Screen pattern from existing feature to model after]

   **Design Options:**
   1. [Option A] - [pros/cons]
   2. [Option B] - [pros/cons]

   **Open Questions:**
   - [Technical uncertainty]
   - [Design decision needed]

   Which approach aligns best with your vision?
   ```

### Step 3: Plan Structure Development

Once aligned on approach:

1. **Create initial plan outline**:
   ```
   Here's my proposed plan structure:

   ## Overview
   [1-2 sentence summary]

   ## Implementation Phases:
   1. [Phase name] - [what it accomplishes]
   2. [Phase name] - [what it accomplishes]
   3. [Phase name] - [what it accomplishes]

   Does this phasing make sense? Should I adjust the order or granularity?
   ```

2. **Get feedback on structure** before writing details

### Step 4: Detailed Plan Writing

After structure approval:

1. **Write the plan** to `thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-description.md`
   - Format: `YYYY-MM-DD-ENG-XXXX-description.md` where:
     - YYYY-MM-DD is today's date
     - ENG-XXXX is the ticket number (omit if no ticket)
     - description is a brief kebab-case description
   - Examples:
     - With ticket: `2025-01-08-ENG-1478-booking-confirmation-screen.md`
     - Without ticket: `2025-01-08-hostel-detail-redesign.md`
2. **Use this template structure**:

````markdown
# [Feature/Task Name] Implementation Plan

## Overview

[Brief description of what we're implementing and why]

## Current State Analysis

[What exists now, what's missing, key constraints discovered]

## Desired End State

[A Specification of the desired end state after this plan is complete, and how to verify it]

### Key Discoveries:
- [Important finding with file:line reference]
- [Pattern to follow]
- [Constraint to work within]

### Reusable Components Identified:
- [Widget from `lib/widgets/` — what it does, where to use it]
- [Theme constant from `lib/base/constants/` — AppColors, AppTextStyles, AppSizes]
- [Existing screen/pattern to model after — file:line reference]

## What We're NOT Doing

[Explicitly list out-of-scope items to prevent scope creep]

## Implementation Approach

[High-level strategy and reasoning]

## Phase 1: [Descriptive Name]

### Overview
[What this phase accomplishes]

### Changes Required:

#### 1. [Component/File Group]
**File**: `path/to/file.ext`
**Changes**: [Summary of changes]

```dart
// Specific code to add/modify
```

### Success Criteria:

#### Automated Verification:
- [ ] Build succeeds: `flutter build apk --debug`
- [ ] Static analysis passes: `flutter analyze`
- [ ] Unit tests pass: `flutter test`
- [ ] No GetX DI errors on app startup

#### Manual Verification:
- [ ] Feature works as expected when tested via UI
- [ ] Screen matches Figma design
- [ ] Responsive sizing works across device sizes
- [ ] No regressions in related features

**Implementation Note**: After completing this phase and all automated verification passes, pause here for manual confirmation from the human that the manual testing was successful before proceeding to the next phase.

---

## Phase 2: [Descriptive Name]

[Similar structure with both automated and manual success criteria...]

---

## Testing Strategy

### Unit Tests:
- [What to test]
- [Key edge cases]

### Widget Tests:
- [Widget rendering tests]
- [Interaction tests]

### Manual Testing Steps:
1. [Specific step to verify feature]
2. [Another verification step]
3. [Edge case to test manually]

## Architecture Checklist

- [ ] Domain layer is pure Dart (no Flutter/GetX/Dio imports)
- [ ] Models extend entities, entities have no fromJson/toJson
- [ ] All repository/use case methods return `Either<Failure, T>`
- [ ] Controllers depend on use cases, not repositories/data sources
- [ ] New bindings registered in correct order in appropriate binding file
- [ ] `Get.lazyPut` uses `fenix: true` where needed
- [ ] New routes added to `routes.dart` and `route_names.dart`
- [ ] All sizing uses `flutter_screenutil` (.w, .h, .sp, .r)
- [ ] Colors from `AppColors`, text styles from constants — no hardcoded values
- [ ] Existing widgets from `lib/widgets/` reused wherever possible
- [ ] Obx wrappers are scoped tightly around reactive state usage
- [ ] Controllers clean up resources in `onClose()`

## Performance Considerations

[Any performance implications or optimizations needed]

## References

- Original ticket: `thoughts/allison/tickets/eng_XXXX.md`
- Related research: `thoughts/shared/research/[relevant].md`
- Similar implementation: `[file:line]`
- Figma design: [link if available]
````

### Step 5: Sync and Review

1. **Sync the thoughts directory**:
   - This ensures the plan is properly indexed and available

2. **Present the draft plan location**:
   ```
   I've created the initial implementation plan at:
   `thoughts/shared/plans/YYYY-MM-DD-ENG-XXXX-description.md`

   Please review it and let me know:
   - Are the phases properly scoped?
   - Are the success criteria specific enough?
   - Any technical details that need adjustment?
   - Missing edge cases or considerations?
   - Have I identified all reusable widgets/themes correctly?
   ```

3. **Iterate based on feedback** - be ready to:
   - Add missing phases
   - Adjust technical approach
   - Clarify success criteria (both automated and manual)
   - Add/remove scope items

4. **Continue refining** until the user is satisfied

## Reuse-First Principle

**CRITICAL: Before proposing ANY new widget, style, color, or UI pattern, you MUST search the existing codebase for something reusable.**

### Mandatory Reuse Checks

Before writing the plan, thoroughly search these locations:

1. **`lib/widgets/`** — Reusable widgets (buttons, cards, inputs, dialogs, loading indicators, etc.)
   - Search for widgets that match or closely match the needed UI
   - If a widget exists but needs minor extension, prefer extending it over creating a new one
   - Document every widget you plan to reuse with its file path

2. **`lib/base/constants/`** — Themes, colors, sizes, text styles, image paths, strings
   - **`AppColors`** — Use existing color constants; never introduce `Color(0xFF...)` or `Colors.xxx`
   - **`AppTextStyles` / text style constants** — Use existing text styles; never create ad-hoc `TextStyle(...)`
   - **`AppSizes` / dimension constants** — Use existing spacing/sizing constants where they exist
   - **`AppImages`** — Use existing image asset references
   - **`AppTexts`** — Use existing string constants for user-visible text

3. **`lib/view/`** — Existing screens that implement similar patterns
   - Find the closest similar screen and model the new implementation after it
   - Copy the same structure for consistency (controller setup, widget tree layout, error handling)

4. **`lib/utils/`** — Extensions and helpers
   - Check for existing extensions on common types (String, DateTime, BuildContext, etc.)
   - Check for existing helper functions before writing new ones

### Reuse Rules

- **DO** reuse existing widgets from `lib/widgets/` — even if they need minor tweaks, prefer modifying them (with backward compatibility) over creating new ones
- **DO** use `AppColors`, `AppTextStyles`, `AppSizes`, and other theme constants throughout
- **DO** follow the exact same UI patterns used in similar existing screens
- **DO** use existing extensions and helpers from `lib/utils/`
- **DON'T** create a new widget when an existing one does 80%+ of what's needed — extend the existing one
- **DON'T** hardcode colors, text styles, dimensions, or strings — always reference constants
- **DON'T** introduce new UI patterns when an established pattern exists in the app
- **DON'T** duplicate helper/utility logic that already exists

### In the Plan

Every phase that touches UI must include a **"Reused Components"** subsection listing:
```markdown
#### Reused Components:
- `lib/widgets/custom_button.dart` — for primary/secondary action buttons
- `AppColors.primaryColor` — for header background
- `AppTextStyles.heading2` — for section titles
- Pattern from `lib/view/booking/booking_screen.dart` — for list layout structure
```

If no existing component can be reused for a particular UI element, explicitly state **why** a new widget is needed and confirm it doesn't duplicate existing functionality.

## Important Guidelines

1. **Be Skeptical**:
   - Question vague requirements
   - Identify potential issues early
   - Ask "why" and "what about"
   - Don't assume - verify with code

2. **Be Interactive**:
   - Don't write the full plan in one shot
   - Get buy-in at each major step
   - Allow course corrections
   - Work collaboratively

3. **Be Thorough**:
   - Read all context files COMPLETELY before planning
   - Research actual code patterns using parallel sub-tasks
   - Include specific file paths and line numbers
   - Write measurable success criteria with clear automated vs manual distinction

4. **Be Practical**:
   - Focus on incremental, testable changes
   - Consider migration and rollback
   - Think about edge cases
   - Include "what we're NOT doing"

5. **Track Progress**:
   - Use TodoWrite to track planning tasks
   - Update todos as you complete research
   - Mark planning tasks complete when done

6. **No Open Questions in Final Plan**:
   - If you encounter open questions during planning, STOP
   - Research or ask for clarification immediately
   - Do NOT write the plan with unresolved questions
   - The implementation plan must be complete and actionable
   - Every decision must be made before finalizing the plan

7. **Respect Architecture**:
   - Ensure Clean Architecture layer boundaries are never violated
   - Domain layer must remain pure Dart
   - Dependencies flow inward only
   - All error handling uses `Either<Failure, T>`
   - GetX patterns followed correctly (DI, state, routing)

## Success Criteria Guidelines

**Always separate success criteria into two categories:**

1. **Automated Verification** (can be run by execution agents):
   - `flutter build apk --debug` succeeds
   - `flutter analyze` passes
   - `flutter test` passes
   - No GetX binding errors on startup
   - Specific files that should exist

2. **Manual Verification** (requires human testing):
   - UI/UX matches Figma design
   - Responsive layout works across device sizes
   - Performance is acceptable (smooth scrolling, fast transitions)
   - Edge cases work correctly
   - User acceptance criteria

## Common Patterns

### For New Screens:
- Add route name in `route_names.dart`
- Add `GetPage` entry in `routes.dart` with binding
- Create controller in `view_model/` depending on use cases
- Create screen widget in `view/` using `GetView<Controller>`
- Register binding in appropriate binding file
- **Reuse existing widgets from `lib/widgets/` wherever possible**
- **Use `AppColors`, `AppTextStyles`, and `flutter_screenutil` for all sizing**

### For New API Features:
- Add endpoint to `data/network/apis.dart`
- Create/update entity in `domain/entities/`
- Create/update model in `data/models/` (extends entity, adds fromJson/toJson)
- Create/update abstract repository in `domain/repositories/`
- Create/update repository impl in `data/repositories/`
- Create/update use case in `domain/use_case/`
- Register all in DI bindings in correct order
- Update controller to call use case and fold result

### For UI Modifications:
- **Search `lib/widgets/` first** for existing components
- **Search `lib/base/constants/` first** for existing styles/colors
- Find the closest existing screen and follow its patterns
- Use `Obx` with tight scope around reactive state
- Use `GetView<Controller>` not StatefulWidget
- All text from `AppTexts`, all colors from `AppColors`
- All sizes via `flutter_screenutil`

## Sub-task Spawning Best Practices

When spawning research sub-tasks:

1. **Spawn multiple tasks in parallel** for efficiency
2. **Each task should be focused** on a specific area
3. **Always include a reuse-discovery task** that searches `lib/widgets/`, `lib/base/constants/`, and similar screens
4. **Provide detailed instructions** including:
   - Exactly what to search for
   - Which directories to focus on
   - What information to extract
   - Expected output format
5. **Be EXTREMELY specific about directories**:
   - Include the full path context in your prompts
6. **Specify read-only tools** to use
7. **Request specific file:line references** in responses
8. **Wait for all tasks to complete** before synthesizing
9. **Verify sub-task results**:
   - If a sub-task returns unexpected results, spawn follow-up tasks
   - Cross-check findings against the actual codebase
   - Don't accept results that seem incorrect
