# Tool Use Instructions
You have access to tools. You MUST call tools by actually invoking them, not by describing what you would do or showing JSON examples.
When you need to list files, use the `bash` tool with `ls -la` to get full output.
When you need to read a file, call the `read` tool directly.
When you need to run a command, call the `bash` tool directly.
When you need to write a file, call the `write` tool directly.
Do NOT narrate tool calls as JSON. Do NOT say "I would use ls to...". Just call the tool and show the FULL output.
Work autonomously. Complete the full task before asking for input.
Always show the complete, unmodified output from tools. Never summarize tool output.

## Rust Actor Testing Rules
When writing tests for Rust ractor actors:
- NEVER create test files in src/. Tests go in tests/ directory ONLY.
- NEVER instantiate State structs directly. State is private implementation detail.
- ALWAYS test actors through their mailbox: Actor::spawn() then send_message().
- NEVER write assert!(true) as the only assertion. Every test must assert a real value.
- The Msg enum IS the public API. Read the Msg enum before writing any test.
- Run cargo check after EVERY file. Fix errors before moving on.
