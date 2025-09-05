# minishell

A small, interactive Unix shell written in C that implements a practical subset of Bash features. This project was built for learning purposes, focusing on parsing, process control, environment management, and signal handling.

- Language: C
- Default branch: `main`
- Status: Work in progress

## Features

Implemented or targeted features typical for a Minishell project:
- Prompt and interactive loop with command history (via GNU Readline)
- Lexer and parser with support for:
  - Quoting: single quotes '…' (literal) and double quotes "…" (with expansion)
  - Pipes | and I/O redirections: >, >>, <, <<
  - Environment variable expansion: $VAR and special $? for last exit status
- Execution:
  - PATH lookup for external programs
  - Pipelines and redirections
  - Proper exit statuses
- Built-ins (no fork when possible):
  - echo (with -n)
  - cd
  - pwd
  - export
  - unset
  - env
  - exit
- Signals (interactive behavior):
  - Ctrl-C (SIGINT) interrupts the current line/child process
  - Ctrl-\ (SIGQUIT) is ignored in the prompt
  - Proper heredoc signal handling

Note: Job control, advanced globbing, and Bash-specific edge cases may not be implemented.

## Build

Prerequisites:
- A C compiler (clang or gcc)
- GNU Readline development headers
- Make

Install Readline:
- macOS (Homebrew):
  - brew install readline
  - You may need to export/include flags so the compiler can find readline:
    - CPPFLAGS="-I$(brew --prefix readline)/include"
    - LDFLAGS="-L$(brew --prefix readline)/lib"
- Debian/Ubuntu:
  - sudo apt-get update && sudo apt-get install -y build-essential libreadline-dev

Build:
- make
- Common targets:
  - make            Build minishell
  - make clean      Remove object files
  - make fclean     Remove objects and binary
  - make re         Rebuild from scratch

The resulting binary is usually named minishell in the project root.

## Run

- ./minishell
- To exit: use exit, Ctrl-D on empty line, or close the terminal.

Examples:
```sh
# Built-ins
pwd
echo -n "Hello, " && echo "world!"
export NAME=Otman
echo "Hi $NAME"
unset NAME
env

# Redirections and pipes
echo "content" > file.txt
cat file.txt | grep con | wc -l
echo "append" >> file.txt
wc -l < file.txt

# Heredoc
cat <<EOF
line 1
line 2
EOF

# External commands via PATH
ls -la | grep minishell
```

## Project Structure

A high-level overview of the repository layout:
- Makefile                Build rules and targets
- minishell.c             Entry point / main shell loop
- minishell.h             Project-wide declarations and types
- free_linked.c           Helpers for freeing linked structures
- env/                    Environment handling (export/unset, list/lookup, $ expansion support)
- exec/                   Execution layer (fork/execve, redirections, pipelines)
- parsing/                Lexer, parser, quotes, tokens, and command/AST building
- libft/                  Custom utility library (string, list, memory, etc.)
- link/                   Linked list or utility helpers shared across modules
- final/                  Integration and/or final orchestration code (naming suggests final assembly)

Note: See source files for authoritative details on each module.

## Signals

- In interactive mode:
  - SIGINT (Ctrl-C): clears current line and returns to prompt
  - SIGQUIT (Ctrl-\): ignored (no “Quit: 3” message)
- In child processes:
  - Signals follow standard Unix behavior; exit statuses are propagated to $?

## Implementation Notes

- Built-ins are executed in the parent process when possible (e.g., cd, export, unset, exit) to affect the current shell state.
- External commands are executed via fork/execve with PATH resolution.
- Heredocs are handled before execution; temporary files or pipes may be used, with careful signal handling.
- The parser respects quoting rules; single quotes are literal, double quotes allow expansion.

## Development

- Code style: idiomatic C, modularized by concern (env, parsing, exec)
- Memory: avoid leaks; free all allocated resources on exit and on parse/exec failures
- Error handling: print messages to stderr and set proper exit codes

Recommended checks:
```sh
# Linux
valgrind --leak-check=full --show-leak-kinds=all ./minishell

# macOS (use leaks for basic checks)
leaks --atExit -- ./minishell
```

## Troubleshooting

- readline not found:
  - Ensure libreadline-dev (Linux) or Homebrew readline (macOS) is installed
  - Add include/lib flags as needed:
    - CPPFLAGS and LDFLAGS pointing to your readline installation
- Odd behavior with signals inside heredoc:
  - Confirm SIGINT handling matches interactive expectations and that temp resources are cleaned up
- PATH or built-ins not working:
  - Verify environment initialization and that built-ins are detected before forking

## Roadmap

- More robust error messages and edge-case handling
- Improved test coverage (parser and heredoc)
- Optional features: wildcard/globbing expansion, advanced variable expansion rules

## License

No license file was found in this repository. If you intend others to use or contribute to this project, consider adding a LICENSE file.

## Acknowledgments

- 42 School Minishell subject
- GNU Readline
- Bash for reference behavior

## Author

- otmansabir — https://github.com/otmansabir
