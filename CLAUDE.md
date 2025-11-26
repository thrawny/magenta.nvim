# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
# Install dependencies
npm ci --production   # for users
npm ci                # for development

# Run tests
npx vitest                        # run all tests
npx vitest filter                 # run tests matching 'filter'
npx vitest -u                     # update snapshots
npx vitest filter --inspect-wait  # debug tests (opens browser debugger)

# Linting and formatting
npx tsc --noEmit      # typecheck
npx eslint . --fix    # lint
npx prettier --write . # format

# Setup pre-commit hooks
npm run setup-precommit
```

## Log Files

- Plugin logs: `/tmp/magenta.log`
- Test logs: `/tmp/test.log`

## Architecture

Magenta is an AI coding assistant plugin for Neovim. It consists of:

1. **Lua side** (`lua/magenta/`): Plugin initialization, keymaps, and Neovim integration. `init.lua` starts the Node process and establishes RPC communication via `NVIM` socket.

2. **Node side** (`node/`): Main application logic using a TEA-inspired architecture (The Elm Architecture).

### TEA Architecture

- **Controllers**: Classes managing state for specific parts of the app (e.g., `Sidebar`, `Chat`, `Thread`)
- **Messages** (`root-msg.ts`): Typed messages that flow through the system triggering state changes
- **Dispatch**: Function passed to controllers for sending messages; central dispatch loop in `magenta.ts`
- **View** (`tea/view.ts`): Declarative VDOM-like rendering using the `d` template literal with `withBindings` for interactivity
- **Render** (`tea/tea.ts`): Manages the rendering cycle

Key principle: If you create a class, you're responsible for passing actions/messages to it.

### Key Files

| File | Purpose |
|------|---------|
| `node/magenta.ts` | Entrypoint, nvim communication, dispatch loop |
| `node/sidebar.ts` | Chat sidebar state, visibility, keybindings |
| `node/chat/chat.ts` | Top-level chat component, manages threads |
| `node/chat/thread.ts` | Message thread, sends messages, tool coordination |
| `node/tools/toolManager.ts` | Tool execution state management |
| `node/providers/provider.ts` | LLM provider abstraction |

### Testing

Tests live in `*.spec.ts` files adjacent to the code they test. Test startup differs from normal startup - see `node/test/preamble.ts` and `node/test/driver.ts` for the test harness.

## LLM Providers

Supports Anthropic, OpenAI, Ollama, Bedrock, and Copilot. Provider abstraction in `node/providers/`.
