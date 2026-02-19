# AI-Assisted Development Workflows for CCC

This document outlines the workflows, sub-agents, and skills designed for AI-driven development of CCC (Claude's C Compiler). These processes are tailored for use with advanced AI tools like **Gemini CLI**, **Claude Code**, and **Google Antigravity**.

## 1. Project Philosophy

CCC is built on a unique "Human-Guided, AI-Executed" philosophy:

*   **Code as Text**: All tasks, bugs, and features are defined in plain text files within the repository (`current_tasks/`, `projects/`, `ideas/`). This ensures clear, persistent context for AI agents.
*   **Zero Dependencies**: The compiler is self-contained (no external lexers, parsers, assemblers, or linkers). This simplifies the environment for AI agents, reducing "it works on my machine" issues.
*   **Modular Architecture**: The codebase is strictly layered (Frontend -> IR -> Passes -> Backend), allowing specialized sub-agents to work in isolation.

## 2. Workflows

### 2.1 Task Resolution Workflow
**Trigger**: A new file appears in `current_tasks/` (e.g., `fix_arm_asm_caspal_instruction.txt`).

1.  **Analyze**:
    *   Agent reads the task file to understand the *Context* (why), *Goal* (what), and *Location* (where).
    *   Agent identifies the "Reproducer" or "Test Case" mentioned in the task.
2.  **Reproduction**:
    *   Agent attempts to compile the specified test case or project to confirm the failure.
    *   *Skill*: `compile_test(test_case_path)`
3.  **Implementation**:
    *   Agent modifies the source code based on the task description.
    *   *Skill*: `modify_code(file, change)`
4.  **Verification**:
    *   Agent recompiles the test case.
    *   Agent verifies the output (e.g., by checking exit code, stdout, or inspecting generated assembly).
    *   *Skill*: `verify_asm(binary_path)`
5.  **Completion**:
    *   Agent updates the task file with status or moves it to `done/` (if such a folder existed) or simply reports completion.

### 2.2 New Project Integration Workflow
**Trigger**: A new entry in `ideas/new_projects.txt`.

1.  **Setup**:
    *   Agent clones the external project source.
    *   Agent configures the build system to use `ccc` (e.g., `export CC=/path/to/ccc`).
2.  **Build Attempt**:
    *   Agent runs the build (e.g., `make`, `cmake`).
    *   *Skill*: `run_build_command(command)`
3.  **Failure Analysis**:
    *   If build fails, Agent captures the error log.
    *   Agent isolates the failure to a minimal C code reproduction.
4.  **Task Creation**:
    *   Agent creates a new file in `current_tasks/` describing the failure (e.g., "Missing support for __builtin_clz").
5.  **Iteration**:
    *   Loop back to **Task Resolution Workflow**.

### 2.3 Code Quality Workflow
**Trigger**: An item in `projects/cleanup_code_quality.txt`.

1.  **Selection**:
    *   Agent picks an item (e.g., "Functions with 10-20+ parameters").
2.  **Refactoring**:
    *   Agent identifies target functions.
    *   Agent introduces a parameter struct or helper function.
    *   *Skill*: `refactor_function_signature(func_name, new_struct)`
3.  **Regression Testing**:
    *   Agent runs the full test suite (or a relevant subset) to ensure no breakage.
    *   *Skill*: `run_all_tests()`
4.  **Status Update**:
    *   Agent marks the item as `[DONE]` in `projects/cleanup_code_quality.txt`.

## 3. Sub-Agents (Personas)

These personas represent specialized roles an AI agent can adopt.

*   **Architect (Role: Strategy)**
    *   **Focus**: `DESIGN_DOC.md`, high-level module interaction.
    *   **Responsibilities**: Deciding on major IR changes, defining new pass pipelines.
    *   **Context**: Needs access to the entire `src/` tree structure and `README.md`.

*   **Frontend Specialist (Role: Parsing)**
    *   **Focus**: `src/frontend/` (Preprocessor, Lexer, Parser, Sema).
    *   **Responsibilities**: Implementing C11/C23 features, fixing macro expansion bugs.
    *   **Context**: Needs `include/` headers and C standard knowledge.

*   **IR Engineer (Role: Optimization)**
    *   **Focus**: `src/ir/` and `src/passes/`.
    *   **Responsibilities**: Implementing SSA optimizations (GVN, LICM), improving register allocation.
    *   **Context**: Needs understanding of dominator trees and data flow analysis.

*   **Backend Expert (Role: Codegen)**
    *   **Focus**: `src/backend/{arch}/`.
    *   **Responsibilities**: Implementing instruction encoding, ABI compliance, peephole optimizations.
    *   **Context**: Needs architecture manuals (Intel SDM, ARM ARM, RISC-V Spec).

*   **QA Lead (Role: Verification)**
    *   **Focus**: `tests/`, `current_tasks/`.
    *   **Responsibilities**: Creating reproducers, running integration tests, validating external project builds.
    *   **Context**: Needs access to build tools (`make`, `cmake`, `qemu`).

## 4. Skills (Capabilities)

These are abstract function calls or capabilities the agents use. For detailed implementation guides and best practices, see the `skills/` directory:

*   [Refactoring Skills](skills/REFACTORING.md)
*   [Performance Optimization Skills](skills/PERFORMANCE.md)
*   [Code Quality Skills](skills/CODE_QUALITY.md)
*   [Testing Skills](skills/TESTING.md)

### Specific Capabilities:

*   **`analyze_task(filepath)`**: Extract "Context", "Goal", and "Files" from a task description.
*   **`compile_test(source_file, target_arch)`**: Run `ccc -o output source.c` for a specific target.
*   **`verify_asm(output_binary, pattern)`**: Disassemble binary and search for specific instructions (e.g., `caspal`).
*   **`run_project_tests(project_dir)`**: Execute the test suite of a compiled project (e.g., `make test`).
*   **`check_code_quality(rule)`**: Scan codebase for violations (e.g., "Find functions with >10 args").
*   **`update_status(file, project, status)`**: meaningful updates to tracking files.

## 5. Tool Integration

### Gemini CLI
*   **Use Case**: Quick context retrieval, simple bug fixes, status checks.
*   **Commands**:
    *   `gemini read_file current_tasks/fix_foo.txt`
    *   `gemini list_files src/backend/arm`
    *   `gemini run "cargo test"`

### Claude Code
*   **Use Case**: Deep reasoning, complex refactoring, architectural implementation.
*   **Commands**:
    *   `claude code --task "Implement CASPAL instruction for ARM based on current_tasks/fix_arm_asm_caspal_instruction.txt"`
    *   `claude code --task "Refactor parse_declaration_rest to use a struct as per projects/cleanup_code_quality.txt"`

### Google Antigravity
*   **Use Case**: Orchestration, long-term memory, multi-step project integration.
*   **Capabilities**:
    *   **Memory**: "Remember that FFmpeg build failed due to missing `__asm__` support in `libavcodec/x86/mathops.h`."
    *   **Planning**: "Break down the implementation of C23 `constexpr` into Frontend, IR, and Backend tasks."
    *   **Orchestration**: "Spawn a **Backend Expert** agent to fix the RISC-V offset bug, then spawn a **QA Lead** to verify the build."
