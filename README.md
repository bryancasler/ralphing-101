# The Complete Ralph Wiggum Guide for macOS

A comprehensive tutorial for autonomous AI-driven development using Claude Code on Mac.

---

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites Setup](#prerequisites-setup)
3. [Project Initialization](#project-initialization)
4. [Define Requirements](#define-requirements)
5. [Planning Mode](#planning-mode)
6. [Building Mode](#building-mode)
7. [Monitoring & Tuning](#monitoring--tuning)
8. [Troubleshooting](#troubleshooting)
9. [Quick Reference](#quick-reference)

---

## Introduction

### What is the Ralph Wiggum Technique?

The Ralph Wiggum technique is an autonomous AI development methodology created by Geoffrey Huntley. It uses a simple bash loop to continuously feed prompts to Claude Code, enabling it to build software iteratively with fresh context each iteration.

**Core Philosophy:**
- Use the AI as a "scheduler" that delegates work to subagents
- Each iteration gets a fresh context window (no pollution)
- One task per iteration, committed before moving on
- Tests and validation provide "backpressure" to ensure quality
- The plan is disposable—regenerate when wrong

**The Loop:**
```bash
while :; do cat PROMPT.md | claude; done
```

That's it. Ralph continuously reads a prompt, executes one task, commits, and repeats with fresh context.

### What We're Building

This tutorial walks through building a **Partner Compatibility Rating App**—a web application where two people can:
- Create/join a shared room with a code
- Rate preferences across categories using sliders
- See each other's progress in real-time
- View compatibility scores

You can adapt this process for any project.

---

## Prerequisites Setup

### Step 1: Install Homebrew

Homebrew is the package manager for macOS. If you don't have it:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the prompts. After installation, add Homebrew to your PATH:

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
source ~/.zshrc
```

Verify installation:

```bash
brew --version
# Homebrew 4.x.x
```

### Step 2: Install Node.js

Claude Code requires Node.js 18 or later.

```bash
# Install Node.js via Homebrew
brew install node

# Verify installation
node --version
# v20.x.x or higher

npm --version
# 10.x.x or higher
```

### Step 3: Install Claude Code CLI

Claude Code is Anthropic's command-line tool for agentic coding.

```bash
# Install globally via npm
npm install -g @anthropic-ai/claude-code

# Verify installation
claude --version
```

### Step 4: Configure API Access

You need an Anthropic API key.

**Get an API Key:**
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Create an account or sign in
3. Navigate to **API Keys** in the sidebar
4. Click **Create Key**
5. Copy the key (starts with `sk-ant-`)

**Set the API Key:**

```bash
# Add to your shell profile for persistence
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.zshrc
source ~/.zshrc

# Verify it's set
echo $ANTHROPIC_API_KEY
# sk-ant-your-key-here
```

**Test Claude Code:**

```bash
claude -p "Say hello in exactly 5 words"
# Hello there, how are you?
```

### Step 5: Install Git

Git is essential for Ralph's commit-per-task workflow.

```bash
# Install via Homebrew
brew install git

# Verify
git --version
# git version 2.x.x
```

**Configure Git identity:**

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Verify
git config --global --list
```

### Step 6: Install Docker Desktop

Ralph requires `--dangerously-skip-permissions` which bypasses all safety prompts. **Always run Ralph in a sandboxed environment.**

**Install Docker Desktop:**

1. Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Open the `.dmg` file
3. Drag Docker to Applications
4. Launch Docker from Applications
5. Complete the setup wizard
6. Wait for Docker to start (whale icon in menu bar)

**Verify Docker:**

```bash
docker --version
# Docker version 24.x.x

docker run hello-world
# Hello from Docker!
```

### Step 7: Create the Ralph Docker Image

Create a sandboxed environment for Ralph:

```bash
# Create a directory for the Docker setup
mkdir -p ~/ralph-docker
cd ~/ralph-docker

# Create the Dockerfile
cat > Dockerfile << 'EOF'
FROM ubuntu:24.04

# Prevent interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Install system dependencies
RUN apt-get update && apt-get install -y \
    curl \
    git \
    ca-certificates \
    gnupg \
    && rm -rf /var/lib/apt/lists/*

# Install Node.js 20.x
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Install Claude Code CLI
RUN npm install -g @anthropic-ai/claude-code

# Install useful development tools
RUN npm install -g serve eslint prettier

# Create non-root user for safety
RUN useradd -m -s /bin/bash ralph
USER ralph

# Set working directory
WORKDIR /workspace

# Configure git for commits inside container
RUN git config --global user.name "Ralph Wiggum" \
    && git config --global user.email "ralph@example.com" \
    && git config --global init.defaultBranch main

# Default command
CMD ["/bin/bash"]
EOF

# Build the image
docker build -t ralph-sandbox .
```

This takes 2-5 minutes. You'll see "Successfully built" when done.

### Step 8: Verify Everything Works

Create a verification script:

```bash
cat > ~/check-ralph-prereqs.sh << 'EOF'
#!/bin/bash

echo "═══════════════════════════════════════════"
echo "  RALPH WIGGUM PREREQUISITES CHECK (macOS)"
echo "═══════════════════════════════════════════"
echo ""

# Colors
GREEN='\033[0;32m'
RED='\033[0;31m'
YELLOW='\033[1;33m'
NC='\033[0m'

check_command() {
    if command -v $1 &> /dev/null; then
        echo -e "${GREEN}✓${NC} $1: $($1 --version 2>/dev/null | head -1)"
    else
        echo -e "${RED}✗${NC} $1: NOT INSTALLED"
    fi
}

# Check each prerequisite
check_command "brew"
check_command "node"
check_command "npm"
check_command "git"
check_command "docker"
check_command "claude"

echo ""

# Check API Key
if [ -n "$ANTHROPIC_API_KEY" ]; then
    echo -e "${GREEN}✓${NC} ANTHROPIC_API_KEY: Set (${ANTHROPIC_API_KEY:0:12}...)"
else
    echo -e "${RED}✗${NC} ANTHROPIC_API_KEY: NOT SET"
fi

# Check Docker running
if docker info &> /dev/null; then
    echo -e "${GREEN}✓${NC} Docker: Running"
else
    echo -e "${RED}✗${NC} Docker: NOT RUNNING (start Docker Desktop)"
fi

# Check Ralph image
if docker image inspect ralph-sandbox &> /dev/null; then
    echo -e "${GREEN}✓${NC} ralph-sandbox image: Built"
else
    echo -e "${YELLOW}!${NC} ralph-sandbox image: Not built (run docker build)"
fi

echo ""
echo "═══════════════════════════════════════════"
EOF

chmod +x ~/check-ralph-prereqs.sh
~/check-ralph-prereqs.sh
```

You should see all green checkmarks. Fix any red items before proceeding.

---

## Project Initialization

### Step 1: Create Project Directory

```bash
# Create project in your preferred location
mkdir -p ~/Projects/compatibility-app
cd ~/Projects/compatibility-app

# Initialize git repository
git init

# Create initial commit
echo "# Compatibility App" > README.md
git add README.md
git commit -m "Initial commit"
```

### Step 2: Create Ralph Directory Structure

```bash
# Create all required directories
mkdir -p specs
mkdir -p src/lib

# Create all required files
touch AGENTS.md
touch IMPLEMENTATION_PLAN.md
touch PROMPT_plan.md
touch PROMPT_build.md
touch loop.sh

# Set permissions
chmod +x loop.sh
```

Your structure now looks like:

```
compatibility-app/
├── loop.sh                    # The outer loop script
├── PROMPT_plan.md             # Planning mode instructions
├── PROMPT_build.md            # Building mode instructions
├── AGENTS.md                  # Operational guide
├── IMPLEMENTATION_PLAN.md     # Task list (Ralph generates this)
├── specs/                     # Requirement specifications
├── src/                       # Application source code
│   └── lib/                   # Shared utilities
└── README.md
```

### Step 3: Create the Loop Script

This is the heart of Ralph—a bash script that continuously feeds prompts to Claude:

```bash
cat > loop.sh << 'SCRIPT'
#!/bin/bash
set -euo pipefail

# ═══════════════════════════════════════════════════════════════════
# RALPH WIGGUM LOOP SCRIPT
# ═══════════════════════════════════════════════════════════════════
# Usage: ./loop.sh [plan] [max_iterations]
# 
# Examples:
#   ./loop.sh              # Build mode, unlimited iterations
#   ./loop.sh 20           # Build mode, max 20 iterations
#   ./loop.sh plan         # Plan mode, unlimited iterations
#   ./loop.sh plan 5       # Plan mode, max 5 iterations
# ═══════════════════════════════════════════════════════════════════

# Color codes
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
CYAN='\033[0;36m'
NC='\033[0m'

# Parse arguments
if [ "${1:-}" = "plan" ]; then
    MODE="plan"
    PROMPT_FILE="PROMPT_plan.md"
    MAX_ITERATIONS=${2:-0}
elif [[ "${1:-}" =~ ^[0-9]+$ ]]; then
    MODE="build"
    PROMPT_FILE="PROMPT_build.md"
    MAX_ITERATIONS=$1
else
    MODE="build"
    PROMPT_FILE="PROMPT_build.md"
    MAX_ITERATIONS=0
fi

ITERATION=0
START_TIME=$(date +%s)

# Get current branch
CURRENT_BRANCH=$(git branch --show-current 2>/dev/null || echo "main")

# ─────────────────────────────────────────────────────────────────
# Header
# ─────────────────────────────────────────────────────────────────
clear
echo -e "${BLUE}═══════════════════════════════════════════════════════════════${NC}"
echo -e "${BLUE}                    RALPH WIGGUM LOOP                           ${NC}"
echo -e "${BLUE}═══════════════════════════════════════════════════════════════${NC}"
echo ""
echo -e "  ${CYAN}Mode:${NC}      ${GREEN}$MODE${NC}"
echo -e "  ${CYAN}Prompt:${NC}    ${YELLOW}$PROMPT_FILE${NC}"
echo -e "  ${CYAN}Branch:${NC}    ${YELLOW}$CURRENT_BRANCH${NC}"
if [ $MAX_ITERATIONS -gt 0 ]; then
    echo -e "  ${CYAN}Max:${NC}       ${YELLOW}$MAX_ITERATIONS iterations${NC}"
else
    echo -e "  ${CYAN}Max:${NC}       ${YELLOW}unlimited${NC} (Ctrl+C to stop)"
fi
echo -e "  ${CYAN}Started:${NC}   $(date '+%Y-%m-%d %H:%M:%S')"
echo ""
echo -e "${BLUE}═══════════════════════════════════════════════════════════════${NC}"

# ─────────────────────────────────────────────────────────────────
# Validation
# ─────────────────────────────────────────────────────────────────

# Check prompt file exists
if [ ! -f "$PROMPT_FILE" ]; then
    echo -e "\n${RED}ERROR: $PROMPT_FILE not found${NC}"
    echo "Create this file before running the loop."
    exit 1
fi

# Check we're in a git repo
if ! git rev-parse --git-dir > /dev/null 2>&1; then
    echo -e "\n${RED}ERROR: Not in a git repository${NC}"
    echo "Run 'git init' first."
    exit 1
fi

# Check Claude is available
if ! command -v claude &> /dev/null; then
    echo -e "\n${RED}ERROR: Claude CLI not found${NC}"
    echo "Install with: npm install -g @anthropic-ai/claude-code"
    exit 1
fi

# Check API key is set
if [ -z "${ANTHROPIC_API_KEY:-}" ]; then
    echo -e "\n${RED}ERROR: ANTHROPIC_API_KEY not set${NC}"
    echo "Export your API key: export ANTHROPIC_API_KEY='sk-ant-...'"
    exit 1
fi

echo -e "\n${GREEN}✓ All checks passed. Starting loop...${NC}\n"
sleep 2

# ─────────────────────────────────────────────────────────────────
# Main Loop
# ─────────────────────────────────────────────────────────────────

while true; do
    # Check iteration limit
    if [ $MAX_ITERATIONS -gt 0 ] && [ $ITERATION -ge $MAX_ITERATIONS ]; then
        echo -e "\n${GREEN}═══════════════════════════════════════════════════════════════${NC}"
        echo -e "${GREEN}  COMPLETE: Reached max iterations ($MAX_ITERATIONS)${NC}"
        echo -e "${GREEN}═══════════════════════════════════════════════════════════════${NC}"
        break
    fi

    ITERATION=$((ITERATION + 1))
    ITER_START=$(date +%s)
    
    echo -e "\n${BLUE}┌─────────────────────────────────────────────────────────────┐${NC}"
    echo -e "${BLUE}│  ITERATION $ITERATION                                              ${NC}"
    echo -e "${BLUE}│  $(date '+%Y-%m-%d %H:%M:%S')                                     ${NC}"
    echo -e "${BLUE}└─────────────────────────────────────────────────────────────┘${NC}\n"

    # ─────────────────────────────────────────────────────────────
    # Run Claude
    # ─────────────────────────────────────────────────────────────
    # Flags:
    #   -p                            : Pipe/headless mode (reads from stdin)
    #   --dangerously-skip-permissions: Auto-approve all tool calls (REQUIRED)
    #   --output-format=stream-json   : Structured output for logging
    #   --model opus                  : Use Opus for best reasoning
    #   --verbose                     : Detailed execution logging
    
    cat "$PROMPT_FILE" | claude -p \
        --dangerously-skip-permissions \
        --output-format=stream-json \
        --model opus \
        --verbose \
        2>&1 || {
            echo -e "\n${YELLOW}⚠ Claude exited with non-zero status${NC}"
        }

    # ─────────────────────────────────────────────────────────────
    # Post-iteration tasks
    # ─────────────────────────────────────────────────────────────
    
    # Try to push changes
    CURRENT_BRANCH=$(git branch --show-current 2>/dev/null || echo "main")
    if git remote get-url origin &> /dev/null; then
        if git push origin "$CURRENT_BRANCH" 2>/dev/null; then
            echo -e "${GREEN}✓ Pushed to origin/$CURRENT_BRANCH${NC}"
        else
            git push -u origin "$CURRENT_BRANCH" 2>/dev/null && \
                echo -e "${GREEN}✓ Created and pushed to origin/$CURRENT_BRANCH${NC}" || \
                echo -e "${YELLOW}⚠ Push skipped (check remote)${NC}"
        fi
    fi

    # Calculate timing
    ITER_END=$(date +%s)
    ITER_DURATION=$((ITER_END - ITER_START))
    TOTAL_DURATION=$((ITER_END - START_TIME))
    
    echo -e "\n${GREEN}✓ Iteration $ITERATION complete (${ITER_DURATION}s, total: ${TOTAL_DURATION}s)${NC}"
    
    # Brief pause between iterations
    sleep 2
done

# ─────────────────────────────────────────────────────────────────
# Summary
# ─────────────────────────────────────────────────────────────────
END_TIME=$(date +%s)
TOTAL_TIME=$((END_TIME - START_TIME))
MINUTES=$((TOTAL_TIME / 60))
SECONDS=$((TOTAL_TIME % 60))

echo -e "\n${BLUE}═══════════════════════════════════════════════════════════════${NC}"
echo -e "${BLUE}  RALPH LOOP FINISHED${NC}"
echo -e "${BLUE}═══════════════════════════════════════════════════════════════${NC}"
echo -e "  ${CYAN}Iterations:${NC}  $ITERATION"
echo -e "  ${CYAN}Duration:${NC}    ${MINUTES}m ${SECONDS}s"
echo -e "  ${CYAN}Commits:${NC}     $(git rev-list --count HEAD 2>/dev/null || echo 'N/A')"
echo -e "${BLUE}═══════════════════════════════════════════════════════════════${NC}"
SCRIPT

chmod +x loop.sh
```

### Step 4: Create AGENTS.md

This operational guide tells Ralph how to build and run the project:

```bash
cat > AGENTS.md << 'EOF'
# Operational Guide

## Project Overview

Partner Compatibility Rating App - A web application for comparing relationship preferences.

## Build & Run

This is a static web application using vanilla HTML/CSS/JavaScript.

**Start Local Server:**
```bash
npx serve src -p 3000
```

**Access:** http://localhost:3000

**Alternative (no server):**
```bash
open src/index.html
```

## Validation Commands

Run these to validate changes before committing:

```bash
# Lint JavaScript (fix issues automatically)
npx eslint src/**/*.js --fix

# Check code formatting
npx prettier --check src/

# Fix formatting
npx prettier --write src/
```

## Tech Stack

- HTML5
- CSS3 (vanilla, no preprocessors)
- JavaScript (ES6+, vanilla, no frameworks)
- LocalStorage for client-side persistence
- No build step required

## Project Structure

```
src/
├── index.html          # Main entry point
├── styles.css          # All styles
├── scripts.js          # Main application logic
└── lib/                # Shared utilities
    └── (utilities)
```

## Codebase Patterns

- **State Management:** Single state object, persist to localStorage
- **DOM Updates:** Use template literals for HTML generation
- **Event Handling:** Delegate events to parent containers
- **Error Handling:** Wrap async operations in try/catch
- **Naming:** camelCase for JS, kebab-case for CSS classes

## Known Issues

(Ralph updates this section as issues are discovered)

---
*Keep this file brief and operational. Status updates belong in IMPLEMENTATION_PLAN.md*
EOF
```

### Step 5: Create Planning Prompt

This prompt tells Ralph how to analyze specs and create an implementation plan:

```bash
cat > PROMPT_plan.md << 'EOF'
0a. Study `specs/*` with up to 250 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md (if present) to understand the plan so far.
0c. Study `src/lib/*` with up to 250 parallel Sonnet subagents to understand shared utilities & components.
0d. For reference, the application source code is in `src/*`.

1. Study @IMPLEMENTATION_PLAN.md (if present; it may be incorrect) and use up to 500 Sonnet subagents to study existing source code in `src/*` and compare it against `specs/*`. Use an Opus subagent to analyze findings, prioritize tasks, and create/update @IMPLEMENTATION_PLAN.md as a bullet point list sorted in priority of items yet to be implemented. Ultrathink. Consider searching for TODO, minimal implementations, placeholders, skipped/flaky tests, and inconsistent patterns. Study @IMPLEMENTATION_PLAN.md to determine starting point for research and keep it up to date with items considered complete/incomplete using subagents.

IMPORTANT: Plan only. Do NOT implement anything. Do NOT assume functionality is missing; confirm with code search first. Treat `src/lib` as the project's standard library for shared utilities and components. Prefer consolidated, idiomatic implementations there over ad-hoc copies.

ULTIMATE GOAL: Build a partner compatibility rating web application where two users can join a shared room, rate their preferences across multiple categories using sliders, see each other's progress in real-time, and view compatibility scores. Consider missing elements and plan accordingly. If an element is missing, search first to confirm it doesn't exist, then if needed author the specification at specs/FILENAME.md. If you create a new element then document the plan to implement it in @IMPLEMENTATION_PLAN.md using a subagent.
EOF
```

### Step 6: Create Building Prompt

This prompt tells Ralph how to implement features from the plan:

```bash
cat > PROMPT_build.md << 'EOF'
0a. Study `specs/*` with up to 500 parallel Sonnet subagents to learn the application specifications.
0b. Study @IMPLEMENTATION_PLAN.md.
0c. For reference, the application source code is in `src/*`.

1. Your task is to implement functionality per the specifications using parallel subagents. Follow @IMPLEMENTATION_PLAN.md and choose the most important item to address. Before making changes, search the codebase (don't assume not implemented) using Sonnet subagents. You may use up to 500 parallel Sonnet subagents for searches/reads and only 1 Sonnet subagent for build/tests. Use Opus subagents when complex reasoning is needed (debugging, architectural decisions).
2. After implementing functionality or resolving problems, run the tests for that unit of code that was improved. If functionality is missing then it's your job to add it as per the application specifications. Ultrathink.
3. When you discover issues, immediately update @IMPLEMENTATION_PLAN.md with your findings using a subagent. When resolved, update and remove the item.
4. When the tests pass, update @IMPLEMENTATION_PLAN.md, then `git add -A` then `git commit` with a message describing the changes. After the commit, `git push`.

99999. Important: When authoring documentation, capture the why — tests and implementation importance.
999999. Important: Single sources of truth, no migrations/adapters. If tests unrelated to your work fail, resolve them as part of the increment.
9999999. As soon as there are no build or test errors create a git tag. If there are no git tags start at 0.0.0 and increment patch by 1 for example 0.0.1 if 0.0.0 does not exist.
99999999. You may add extra logging if required to debug issues.
999999999. Keep @IMPLEMENTATION_PLAN.md current with learnings using a subagent — future work depends on this to avoid duplicating efforts. Update especially after finishing your turn.
9999999999. When you learn something new about how to run the application, update @AGENTS.md using a subagent but keep it brief.
99999999999. For any bugs you notice, resolve them or document them in @IMPLEMENTATION_PLAN.md using a subagent even if it is unrelated to the current piece of work.
999999999999. Implement functionality completely. Placeholders and stubs waste efforts and time redoing the same work.
9999999999999. When @IMPLEMENTATION_PLAN.md becomes large periodically clean out the items that are completed from the file using a subagent.
99999999999999. If you find inconsistencies in the specs/* then use an Opus 4.5 subagent with 'ultrathink' requested to update the specs.
999999999999999. IMPORTANT: Keep @AGENTS.md operational only — status updates and progress notes belong in IMPLEMENTATION_PLAN.md. A bloated AGENTS.md pollutes every future loop's context.
EOF
```

### Step 7: Create Docker Run Script

Create a convenient script to run Ralph in Docker:

```bash
cat > run-ralph.sh << 'SCRIPT'
#!/bin/bash

# ═══════════════════════════════════════════════════════════════════
# RUN RALPH IN DOCKER SANDBOX
# ═══════════════════════════════════════════════════════════════════
# Usage: ./run-ralph.sh [plan|build] [max_iterations]
#
# Examples:
#   ./run-ralph.sh              # Enter container shell
#   ./run-ralph.sh plan 3       # Run planning mode
#   ./run-ralph.sh build 20     # Run build mode
# ═══════════════════════════════════════════════════════════════════

# Check Docker is running
if ! docker info &> /dev/null; then
    echo "ERROR: Docker is not running. Start Docker Desktop first."
    exit 1
fi

# Check API key
if [ -z "$ANTHROPIC_API_KEY" ]; then
    echo "ERROR: ANTHROPIC_API_KEY not set"
    echo "Run: export ANTHROPIC_API_KEY='sk-ant-...'"
    exit 1
fi

# Get the project directory (where this script is)
PROJECT_DIR="$(cd "$(dirname "$0")" && pwd)"

echo "═══════════════════════════════════════════════════════════════"
echo "  RALPH WIGGUM DOCKER SANDBOX"
echo "═══════════════════════════════════════════════════════════════"
echo "  Project: $PROJECT_DIR"
echo "  API Key: ${ANTHROPIC_API_KEY:0:12}..."
echo "═══════════════════════════════════════════════════════════════"

# Determine command to run
if [ $# -eq 0 ]; then
    # No arguments - start interactive shell
    echo "  Mode: Interactive shell"
    echo "═══════════════════════════════════════════════════════════════"
    echo ""
    echo "Inside the container, run:"
    echo "  ./loop.sh plan 3     # Generate implementation plan"
    echo "  ./loop.sh 20         # Build with max 20 iterations"
    echo "  ./loop.sh            # Build unlimited (Ctrl+C to stop)"
    echo ""
    
    docker run -it --rm \
        -v "$PROJECT_DIR:/workspace" \
        -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
        -w /workspace \
        ralph-sandbox \
        /bin/bash
else
    # Arguments provided - run loop directly
    MODE="${1:-build}"
    ITERATIONS="${2:-0}"
    
    if [ "$MODE" = "plan" ]; then
        echo "  Mode: Planning ($ITERATIONS iterations)"
        CMD="./loop.sh plan $ITERATIONS"
    else
        echo "  Mode: Building ($ITERATIONS iterations)"
        CMD="./loop.sh $ITERATIONS"
    fi
    echo "═══════════════════════════════════════════════════════════════"
    echo ""
    
    docker run -it --rm \
        -v "$PROJECT_DIR:/workspace" \
        -e ANTHROPIC_API_KEY="$ANTHROPIC_API_KEY" \
        -w /workspace \
        ralph-sandbox \
        /bin/bash -c "$CMD"
fi
SCRIPT

chmod +x run-ralph.sh
```

### Step 8: Commit the Setup

```bash
git add -A
git commit -m "Setup Ralph Wiggum project structure"
```

---

## Define Requirements

This phase happens **outside** the Ralph loop. You can do it manually or with Claude's help in a regular conversation.

### Step 1: Identify Your Job to Be Done (JTBD)

The JTBD is the high-level outcome users want:

> "Help partners discover and compare their preferences to understand compatibility"

### Step 2: Break into Topics of Concern

Each topic becomes one spec file. Use the "one sentence without 'and'" test—if you need "and" to describe it, split it into multiple topics.

| Topic | Description | Spec File |
|-------|-------------|-----------|
| Room Management | Create/join rooms with codes | `specs/room-management.md` |
| Preference System | Categories and slider ratings | `specs/preference-system.md` |
| Real-time Sync | Live updates between partners | `specs/realtime-sync.md` |
| Scoring | Compatibility calculations | `specs/scoring.md` |
| User Interface | Layout, responsiveness, UX | `specs/user-interface.md` |

### Step 3: Write Specification Files

Create each spec with clear requirements and acceptance criteria:

**specs/room-management.md:**

```bash
cat > specs/room-management.md << 'EOF'
# Room Management

## Overview

Users create or join rooms using short codes to rate preferences together.

## Requirements

### Room Creation
- Generate unique 6-character alphanumeric room codes
- Exclude confusing characters (0/O, 1/l/I) for verbal sharing
- Creator automatically joins as "Partner 1"
- Display room code prominently for sharing
- Room persists for 24 hours

### Room Joining
- Input field for entering room code
- Validate code exists before joining
- Joiner becomes "Partner 2"
- Maximum 2 users per room
- Show error for invalid/full rooms

### Room State
- Store in localStorage: room code, partner IDs, creation time
- Generate unique partner ID on first visit (persist in localStorage)
- Handle page refresh gracefully (restore session)
- Show "waiting for partner" until both joined

### Room Lifecycle
- Active: both partners present
- Waiting: one partner, waiting for other
- Expired: 24 hours elapsed
- Complete: both finished all ratings

## Acceptance Criteria

- [ ] Room codes are 6 characters using safe alphanumeric set
- [ ] Creating room shows code prominently with copy button
- [ ] Joining valid code shows partner's presence
- [ ] Joining invalid code shows helpful error message
- [ ] Refreshing page maintains room session
- [ ] Third user attempting to join sees "room full" error
- [ ] Room state persists across browser sessions
EOF
```

**specs/preference-system.md:**

```bash
cat > specs/preference-system.md << 'EOF'
# Preference System

## Overview

Categories of relationship preferences with slider-based ratings from 1-10.

## Requirements

### Categories

Six preference categories, each with 5-8 items:

1. **Communication**
   - "I prefer to resolve conflicts immediately vs. taking time to cool off"
   - "I express love through words vs. actions"
   - "I need lots of daily check-ins vs. independence"
   - "I prefer direct honesty vs. tactful delivery"
   - "I share problems to vent vs. seeking solutions"

2. **Lifestyle**
   - "I'm an early bird vs. night owl"
   - "I prefer a tidy space vs. comfortable clutter"
   - "I like routine vs. spontaneity"
   - "I'm introverted vs. extroverted"
   - "I prioritize work vs. personal time"

3. **Adventure**
   - "I enjoy trying new foods vs. sticking to favorites"
   - "I prefer staycations vs. travel adventures"
   - "I'm risk-averse vs. thrill-seeking"
   - "I like planned activities vs. going with the flow"
   - "I prefer familiar places vs. exploring new ones"

4. **Finance**
   - "I'm a saver vs. spender"
   - "I prefer separate vs. joint finances"
   - "I value experiences vs. material things"
   - "I'm comfortable with debt vs. debt-averse"
   - "I prefer budgeting vs. flexible spending"

5. **Family**
   - "I want children vs. prefer not to have children"
   - "I value extended family involvement vs. independence"
   - "I prefer traditional roles vs. equal partnership"
   - "Holidays with family vs. creating own traditions"
   - "I want pets vs. prefer no pets"

6. **Intimacy**
   - "I need lots of physical affection vs. personal space"
   - "I prefer quality time together vs. parallel activities"
   - "I express feelings openly vs. privately"
   - "I need frequent reassurance vs. assumed security"
   - "I value deep conversations vs. comfortable silence"

### Slider Interface
- Range: 1 to 10
- Show numeric value while dragging
- Labels at endpoints (1 = left statement, 10 = right statement)
- Smooth animation on touch and mouse
- Default to middle (5) before user interaction

### Rating Flow
- Navigate through categories sequentially
- Must complete all items in category before next
- Can go back to previous categories
- Progress indicator shows completion
- Partner's ratings hidden until both complete category

## Acceptance Criteria

- [ ] All 6 categories present with 5+ items each
- [ ] Sliders work on mouse and touch devices
- [ ] Current value displays while adjusting
- [ ] Endpoint labels clearly describe the scale
- [ ] Progress shows X/Y items per category
- [ ] Cannot skip items within a category
- [ ] Can navigate between completed categories
- [ ] Partner ratings hidden until mutual completion
EOF
```

**specs/realtime-sync.md:**

```bash
cat > specs/realtime-sync.md << 'EOF'
# Real-time Synchronization

## Overview

Both partners see updates as the other rates preferences.

## Requirements

### Sync Mechanism
- Poll-based synchronization (no external services)
- Poll interval: 2 seconds
- Use localStorage as shared state (same browser) or simple JSON file

### For Demo/Local Testing
Since this is a client-side app without a backend:
- Support "same browser, different tabs" testing via localStorage events
- Storage event listener for cross-tab communication
- Each tab represents one partner

### Data to Sync
- Partner presence (online/last seen)
- Category completion status per partner
- Individual ratings (revealed after both complete category)
- Overall progress percentage

### Connection Status
- Show indicator: "Partner online" / "Partner away"
- Update presence on any activity
- Mark as "away" after 30 seconds of inactivity

### State Management
- Single source of truth in localStorage
- Key structure: `room:{roomCode}:state`
- Include timestamps for all changes
- Last-write-wins for conflicts

## Acceptance Criteria

- [ ] Partner's progress updates within 3 seconds
- [ ] Connection status clearly visible
- [ ] Works across browser tabs (same browser)
- [ ] Presence indicator reflects partner activity
- [ ] No data loss on rapid concurrent edits
- [ ] State survives page refresh
EOF
```

**specs/scoring.md:**

```bash
cat > specs/scoring.md << 'EOF'
# Compatibility Scoring

## Overview

Calculate and display compatibility based on preference matches.

## Requirements

### Scoring Algorithm

**Per-item compatibility:**
```
itemScore = 10 - |partner1Rating - partner2Rating|
```
- Maximum: 10 (identical ratings)
- Minimum: 1 (opposite ratings: 1 vs 10)

**Per-category score:**
```
categoryScore = average(itemScores) / 10 * 100
```
- Expressed as percentage (0-100%)

**Overall score:**
```
overallScore = average(categoryScores)
```
- Equal weight to all categories
- Expressed as percentage

### Score Display

**Category Results (after both complete):**
- Percentage with one decimal (e.g., "78.5%")
- Color coding:
  - Green: 80%+ (highly compatible)
  - Yellow: 50-79% (some differences)
  - Red: <50% (significant differences)
- Bar chart showing each category

**Overall Score:**
- Large, prominent display
- Same color coding as categories
- Show after at least one category complete

**Item Comparison (detail view):**
- Show both partners' ratings side by side
- Highlight matches (within 2 points)
- Highlight mismatches (5+ points apart)

### Progressive Reveal
- Show category score when both complete that category
- Update overall score as categories complete
- Full results available when all categories done

## Acceptance Criteria

- [ ] Scores calculate correctly per algorithm
- [ ] Category scores appear after both partners complete
- [ ] Color coding matches specified thresholds
- [ ] Overall score updates as categories complete
- [ ] Item comparison shows both ratings
- [ ] Matches and mismatches visually distinct
- [ ] Scores persist across sessions
EOF
```

**specs/user-interface.md:**

```bash
cat > specs/user-interface.md << 'EOF'
# User Interface

## Overview

Clean, mobile-first interface for the compatibility rating app.

## Requirements

### Screen States

1. **Landing Screen**
   - App title and brief description
   - "Create Room" button (primary action)
   - "Join Room" button (secondary action)
   - Room code input field (for joining)

2. **Waiting Screen**
   - Display room code (large, copyable)
   - "Share this code with your partner"
   - Copy to clipboard button
   - Waiting animation/indicator
   - Cancel/leave room option

3. **Rating Screen**
   - Category name as header
   - Progress indicator (e.g., "Question 3 of 5")
   - Current preference item text
   - Slider with value display
   - Previous/Next navigation
   - Partner status indicator (corner)

4. **Results Screen**
   - Overall compatibility score (hero)
   - Category breakdown (bar chart or cards)
   - Expandable item details per category
   - "Start Over" option
   - Share results option (copy text)

### Layout Principles
- Mobile-first (320px minimum width)
- Single column on mobile
- Centered content, max-width 600px
- Comfortable touch targets (44px minimum)
- Clear visual hierarchy

### Visual Design
- Soft, friendly color palette
- Primary: Warm purple (#7C3AED)
- Success: Green (#10B981)
- Warning: Amber (#F59E0B)
- Error: Red (#EF4444)
- Background: Light gray (#F9FAFB)
- Clean, readable typography
- Subtle shadows and rounded corners

### Animations
- Smooth transitions between screens (fade)
- Slider value changes animated
- Progress bar fills smoothly
- Score reveals with subtle animation

### Accessibility
- Semantic HTML structure
- ARIA labels for interactive elements
- Keyboard navigation support
- Focus indicators visible
- Color not sole indicator (use icons/text too)
- Minimum contrast ratios met

### Error States
- Invalid room code: inline message
- Network issues: toast notification
- Empty required fields: highlight field
- All errors include recovery action

## Acceptance Criteria

- [ ] Works on 320px screens and up
- [ ] All touch targets 44px minimum
- [ ] Keyboard can navigate entire app
- [ ] Screen reader can understand all content
- [ ] Colors match specified palette
- [ ] Transitions feel smooth (no jank)
- [ ] Error messages explain how to recover
- [ ] Loading states prevent confusion
EOF
```

### Step 4: Commit Specifications

```bash
git add specs/
git commit -m "Add requirement specifications"
```

---

## Planning Mode

Now we use Ralph to generate an implementation plan.

### Step 1: Enter Docker Sandbox

```bash
./run-ralph.sh
```

This opens an interactive shell inside the Docker container with your project mounted.

### Step 2: Run Planning Loop

Inside the container:

```bash
./loop.sh plan 3
```

This runs up to 3 planning iterations. Ralph will:
1. Read all spec files in `specs/`
2. Analyze existing code in `src/` (empty initially)
3. Compare specs vs. code (gap analysis)
4. Generate `IMPLEMENTATION_PLAN.md` with prioritized tasks

### Step 3: Review the Plan

After planning completes, review the generated plan:

```bash
cat IMPLEMENTATION_PLAN.md
```

**Good plan characteristics:**
- Logical ordering (foundations first)
- Tasks are appropriately sized (not too big)
- All spec requirements covered
- No duplicate tasks
- Clear task descriptions

**Example plan structure:**
```markdown
# Implementation Plan

## High Priority

- [ ] Create basic HTML structure with all screen containers
- [ ] Implement room code generation (6 chars, safe alphabet)
- [ ] Create room state management in localStorage
- [ ] Build landing screen UI (create/join buttons)
- [ ] Build waiting screen UI (room code display, copy button)

## Medium Priority

- [ ] Implement preference data structure (categories, items)
- [ ] Build slider component with value display
- [ ] Create rating screen layout
- [ ] Implement category navigation

## Lower Priority

- [ ] Add scoring algorithm
- [ ] Build results display
- [ ] Implement cross-tab sync via storage events
- [ ] Add animations and transitions
```

### Step 4: Regenerate If Needed

If the plan looks wrong:

```bash
rm IMPLEMENTATION_PLAN.md
./loop.sh plan 3
```

You can also edit specs for clarity, then regenerate.

---

## Building Mode

Now Ralph implements the application.

### Step 1: Start the Build Loop

Inside the Docker container:

```bash
# Run 20 iterations
./loop.sh 20

# Or unlimited (stop with Ctrl+C)
./loop.sh
```

### Step 2: What Happens Each Iteration

Each iteration, Ralph:

1. **Orients** — Reads specs and AGENTS.md
2. **Studies Plan** — Reads IMPLEMENTATION_PLAN.md
3. **Selects** — Picks the most important incomplete task
4. **Investigates** — Searches codebase (never assumes)
5. **Implements** — Writes code using subagents
6. **Validates** — Runs linting/tests (backpressure)
7. **Updates** — Marks task done, notes discoveries
8. **Commits** — Creates git commit
9. **Clears** — Context window resets for next iteration

### Step 3: Monitor Progress

Open a new terminal tab to monitor:

```bash
cd ~/Projects/compatibility-app

# Watch git log
watch -n 5 'git log --oneline -15'

# Check implementation plan status
cat IMPLEMENTATION_PLAN.md

# View the app (outside Docker)
open src/index.html
# Or
npx serve src -p 3000
```

### Step 4: When to Stop

Stop the loop (`Ctrl+C`) when:
- All tasks in IMPLEMENTATION_PLAN.md are done
- The app works as specified
- You want to manually review/adjust

---

## Monitoring & Tuning

### Common Issues and Solutions

| Issue | Symptom | Solution |
|-------|---------|----------|
| **Duplicate code** | Same feature implemented twice | Add guardrail: "Search with 3+ subagents before implementing" |
| **Skipping validation** | Commits without linting | Strengthen backpressure: "MANDATORY: Run eslint before commit" |
| **Going in circles** | Same task attempted repeatedly | Delete plan and regenerate |
| **Wrong priorities** | Nice-to-haves before core features | Adjust spec priorities or regenerate plan |
| **Large commits** | Multiple features per commit | Add: "Implement ONE task, then commit immediately" |

### Adding Guardrails

When Ralph fails repeatedly, add guardrails to `PROMPT_build.md`:

```bash
# Open the file
nano PROMPT_build.md

# Add after the existing 9s at the bottom:

9999999999999999. CRITICAL: Before implementing ANY feature, use at minimum 3 subagents to search for existing implementations. Report findings before writing code.

99999999999999999. MANDATORY: Run 'npx eslint src/' before every commit. Fix all errors. No commits with lint errors.

999999999999999999. ONE task per iteration. After completing ONE task, commit and exit. Do not continue to other tasks.
```

### Regenerating the Plan

Regenerate when:
- Ralph implements wrong things
- Plan is cluttered with completed items
- You've changed specifications
- You're confused about state

```bash
rm IMPLEMENTATION_PLAN.md
./loop.sh plan 3
```

### Escape Hatches

| Command | Purpose |
|---------|---------|
| `Ctrl+C` | Stop the loop immediately |
| `git reset --hard` | Revert all uncommitted changes |
| `git reset --hard HEAD~1` | Undo the last commit |
| `rm IMPLEMENTATION_PLAN.md` | Force fresh planning |
| `git stash` | Temporarily save changes |
| `exit` | Leave Docker container |

---

## Troubleshooting

### Docker Issues

**"Cannot connect to Docker daemon"**
```bash
# Start Docker Desktop from Applications
open -a Docker

# Wait for it to start, then verify
docker info
```

**"Image not found"**
```bash
# Rebuild the image
cd ~/ralph-docker
docker build -t ralph-sandbox .
```

**"Permission denied" in container**
```bash
# Files created by Docker may have wrong permissions
# Outside container, fix with:
sudo chown -R $(whoami) ~/Projects/compatibility-app
```

### Claude Issues

**"ANTHROPIC_API_KEY not set"**
```bash
# Set it for current session
export ANTHROPIC_API_KEY="sk-ant-your-key"

# Or add to shell profile
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key"' >> ~/.zshrc
source ~/.zshrc
```

**"Rate limited" or API errors**
```bash
# Wait a moment and retry
sleep 60
./loop.sh 5
```

**Claude seems stuck or unresponsive**
```bash
# Stop with Ctrl+C
# Check your API key is valid
claude -p "Say hello"

# If that works, restart the loop
./loop.sh 5
```

### Git Issues

**"Not a git repository"**
```bash
git init
git add -A
git commit -m "Initial commit"
```

**"Push rejected"**
```bash
# If you don't have a remote, skip push or add one:
git remote add origin https://github.com/you/repo.git
git push -u origin main
```

**"Merge conflicts"**
```bash
# In the container or locally
git status
# Edit conflicted files
git add -A
git commit -m "Resolve merge conflicts"
```

### Loop Issues

**Ralph keeps doing the same task**
```bash
# The plan may be stuck - regenerate it
rm IMPLEMENTATION_PLAN.md
./loop.sh plan 3
```

**Ralph creates broken code**
```bash
# Revert last commit
git reset --hard HEAD~1

# Add more specific guardrails to PROMPT_build.md
# Then retry
./loop.sh 5
```

**Loop exits immediately**
```bash
# Check prompt file exists
cat PROMPT_build.md

# Check for syntax errors in loop.sh
bash -n loop.sh

# Run claude manually to test
cat PROMPT_build.md | claude -p --verbose
```

---

## Quick Reference

### File Structure

```
compatibility-app/
├── run-ralph.sh               # Docker run helper
├── loop.sh                    # Ralph loop script
├── PROMPT_plan.md             # Planning mode prompt
├── PROMPT_build.md            # Building mode prompt
├── AGENTS.md                  # Operational guide
├── IMPLEMENTATION_PLAN.md     # Task list (Ralph-managed)
├── specs/                     # Requirement specifications
│   ├── room-management.md
│   ├── preference-system.md
│   ├── realtime-sync.md
│   ├── scoring.md
│   └── user-interface.md
├── src/                       # Application source code
│   ├── index.html
│   ├── styles.css
│   ├── scripts.js
│   └── lib/
└── README.md
```

### Commands

```bash
# ═══════════════════════════════════════════
# DOCKER COMMANDS
# ═══════════════════════════════════════════

# Enter sandbox (interactive shell)
./run-ralph.sh

# Run planning directly
./run-ralph.sh plan 3

# Run building directly
./run-ralph.sh build 20

# ═══════════════════════════════════════════
# INSIDE CONTAINER
# ═══════════════════════════════════════════

# Planning mode
./loop.sh plan              # Unlimited
./loop.sh plan 3            # Max 3 iterations

# Building mode
./loop.sh                   # Unlimited (Ctrl+C to stop)
./loop.sh 20                # Max 20 iterations

# Exit container
exit

# ═══════════════════════════════════════════
# EMERGENCY COMMANDS
# ═══════════════════════════════════════════

# Stop loop
Ctrl+C

# Revert uncommitted changes
git reset --hard

# Undo last commit
git reset --hard HEAD~1

# Regenerate plan
rm IMPLEMENTATION_PLAN.md && ./loop.sh plan 3

# View progress
cat IMPLEMENTATION_PLAN.md
git log --oneline -20
```

### Key Principles

| Principle | What It Means |
|-----------|---------------|
| **Context is precious** | Keep prompts tight; delegate to subagents |
| **Plan is disposable** | Regenerate when wrong—it's cheap |
| **Don't assume** | Always search before implementing |
| **Backpressure is critical** | Tests/lints must pass before commit |
| **Let Ralph Ralph** | Trust iteration for eventual consistency |
| **Use protection** | Always run in Docker sandbox |
| **Tune like a guitar** | Observe failures, add guardrails |

### Workflow Summary

```
┌─────────────────────────────────────────────────────────────┐
│  1. PREREQUISITES                                           │
│     Install: Homebrew, Node.js, Claude Code, Git, Docker    │
│     Configure: API key, Git identity, Docker image          │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  2. PROJECT SETUP                                           │
│     Create: directories, loop.sh, prompts, AGENTS.md        │
│     Run: ./run-ralph.sh to enter sandbox                    │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  3. DEFINE REQUIREMENTS                                     │
│     Write: specs/*.md (one per topic of concern)            │
│     Include: requirements + acceptance criteria             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  4. PLANNING MODE                                           │
│     Run: ./loop.sh plan 3                                   │
│     Output: IMPLEMENTATION_PLAN.md                          │
│     Review: regenerate if wrong                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  5. BUILDING MODE                                           │
│     Run: ./loop.sh 20                                       │
│     Each iteration: select → implement → test → commit      │
│     Monitor: git log, IMPLEMENTATION_PLAN.md                │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  6. MONITOR & TUNE                                          │
│     Watch: for repeated failures                            │
│     Add: guardrails to prompts                              │
│     Regenerate: plan when stale                             │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  7. COMPLETE                                                │
│     All tasks done, app works as specified                  │
│     Final testing and deployment                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Next Steps

Once your app is working:

1. **Test thoroughly** — Run through all user flows manually
2. **Deploy** — Push to GitHub Pages, Netlify, or Vercel
3. **Iterate** — Add new specs, regenerate plan, build more features
4. **Share** — Let others try your compatibility app!

The Ralph Wiggum technique scales to larger projects—just add more specs and let Ralph work through the plan. The key is clear specifications and trusting the iteration process.

Happy building! 🚀
