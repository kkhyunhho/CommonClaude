<!-- Appended after the Python table in §2 Naming ONLY when the project
     includes C code. -->

C conventions (Google C++ style guide, C-adapted; third-party code exempt):

| Element                      | Style                  | Example            |
|------------------------------|------------------------|--------------------|
| Variable / parameter / local | `snake_case`           | `can_id`           |
| Function                     | `snake_case` (`module_action`) | `motor_cmd_jog_start` |
| Struct / typedef             | `snake_case_t`         | `btn_ctx_t`        |
| Enum constant                | `SCREAMING_SNAKE_CASE` | `AXIS_Z`           |
| Macro / `#define`            | `SCREAMING_SNAKE_CASE` | `MKS_STATUS_OK`    |
| File                         | `snake_case.c` / `.h`  | `motor_cmd.c`      |
| Module-internal globals      | `static`, prefix `s_`  | `static bool s_jogging` |

C rules: `#pragma once` guards; K&R braces (opening brace same line for
`if`/`for`/`while`, separate line for function definitions); always brace
single-statement bodies; declare variables in the narrowest scope. Lint C
with `clang-format` (LLVM, 80-col) and `cppcheck --enable=warning,style`.
