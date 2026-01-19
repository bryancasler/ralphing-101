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
7. [Deployment](#deployment)
8. [Monitoring & Tuning](#monitoring--tuning)
9. [Troubleshooting](#troubleshooting)
10. [Quick Reference](#quick-reference)

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

### Key Terminology

Before diving in, here are terms you'll encounter:

| Term | Meaning |
|------|---------|
| **JTBD** | "Job to Be Done" — the high-level outcome users want |
| **Spec** | A specification file describing requirements for one feature area |
| **Subagent** | Claude spawning a separate task to do work in parallel |
| **Sonnet** | A faster, cheaper Claude model (good for simple tasks like reading files) |
| **Opus** | The most capable Claude model (good for complex reasoning) |
| **Ultrathink** | Instruction telling Claude to engage in deeper, more thorough reasoning |
| **Backpressure** | Validation (tests, linting) that rejects bad code before committing |
| **Context window** | The ~200K tokens Claude can "see" at once |

### What We're Building

This tutorial walks through building a **Partner Compatibility Rating App**—a web application where two people can:
- Create/join a shared room with a code
- Rate preferences across categories using sliders
- See each other's progress in real-time
- View compatibility scores

You can adapt this process for any project.

### Cost Expectations

**Important:** Claude API usage costs money. Approximate costs:

| Model | Input | Output |
|-------|-------|--------|
| Opus | $15 / million tokens | $75 / million tokens |
| Sonnet | $3 / million tokens | $15 / million tokens |

**Rough estimate:** Building a small app with Ralph might cost $5-50 depending on complexity and iterations.

**To monitor spending:**
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Click **Usage** in the sidebar
3. Set up usage alerts under **Settings** → **Limits**

**Tip:** Set a spending limit (e.g., $20) to avoid surprises.

### What Claude Desktop Can and Cannot Do

Throughout this tutorial, you'll see two types of steps:

| Icon | Meaning |
|------|---------|
| 🤖 | **Claude Desktop can do this** — Copy the provided prompt into Claude Desktop |
| 👤 | **Human required** — You must do this manually (account creation, authentication, etc.) |

Claude Desktop cannot perform steps that require:
- Creating accounts (CAPTCHA, email verification)
- Entering payment information
- OAuth authorization flows
- Installing desktop applications (macOS permission dialogs)
- Running interactive Docker sessions

---

## Prerequisites Setup

### Step 1: Install Homebrew

👤 **Human Required**

Homebrew is the package manager for macOS. Open Terminal and run:

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

*Note: Claude Desktop cannot run this because it requires interactive Terminal input and may prompt for your password.*

---

### Step 2: Install Node.js and Git

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Install Node.js and Git on my Mac using Homebrew. After installation, verify both are working by checking their versions. Show me the output of the version checks.
```

**What this does:**
- Runs `brew install node git`
- Verifies with `node --version` and `git --version`

---

### Step 3: Install Claude Code CLI

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Install the Claude Code CLI globally using npm. After installation, verify it's working by checking the version. Show me the output.
```

**What this does:**
- Runs `npm install -g @anthropic-ai/claude-code`
- Verifies with `claude --version`

---

### Step 4: Create Anthropic Account and Get API Key

👤 **Human Required**

**Create Account:**
1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Click "Sign up"
3. Enter your email and create a password
4. Verify your email address
5. Add a payment method (required for API access)

**Get API Key:**
1. In the console, click **API Keys** in the sidebar
2. Click **Create Key**
3. Give it a name like "Ralph on Mac"
4. Copy the key (starts with `sk-ant-`)—you won't see it again

**Set a Spending Limit (Recommended):**
1. In console.anthropic.com, go to **Settings** → **Limits**
2. Set a monthly limit (e.g., $20 or $50)
3. Enable email alerts for usage thresholds

*Note: Claude Desktop cannot create accounts or access authenticated dashboards.*

---

### Step 5: Set API Key in Shell

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop (replace `YOUR_API_KEY` with your actual key):

```
Add my Anthropic API key to my shell configuration so it persists across terminal sessions. My API key is: sk-ant-xxxxx (replace with actual key)

Add it to ~/.zshrc and reload the shell configuration. Then verify the key is set by echoing the first 15 characters (for security, don't show the full key).
```

**What this does:**
- Adds `export ANTHROPIC_API_KEY="sk-ant-..."` to `~/.zshrc`
- Runs `source ~/.zshrc`
- Verifies with a partial echo

---

### Step 6: Configure Git Identity

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop (replace with your info):

```
Configure Git with my identity:
- Name: Your Full Name
- Email: your.email@example.com

Also enable the macOS Keychain credential helper so Git remembers my GitHub credentials. Show me the final git config to confirm.
```

**What this does:**
- Runs `git config --global user.name "..."`
- Runs `git config --global user.email "..."`
- Runs `git config --global credential.helper osxkeychain`

---

### Step 7: Create GitHub Account

👤 **Human Required**

If you don't have a GitHub account:

1. Go to [github.com](https://github.com)
2. Click **Sign up**
3. Enter your email address
4. Create a password
5. Choose a username
6. Complete the verification puzzle
7. Confirm your email address

*Note: Claude Desktop cannot create accounts due to CAPTCHA and email verification.*

---

### Step 8: Create GitHub Personal Access Token

👤 **Human Required**

GitHub no longer accepts passwords for Git operations. You need a Personal Access Token:

1. Go to GitHub → Click your profile photo → **Settings**
2. Scroll down the left sidebar to **Developer settings** (at the bottom)
3. Click **Personal access tokens** → **Tokens (classic)**
4. Click **Generate new token** → **Generate new token (classic)**
5. Give it a name like "Ralph on Mac"
6. Set expiration (90 days, or "No expiration" for convenience)
7. Check the **repo** scope (full control of private repositories)
8. Click **Generate token**
9. **Copy the token immediately**—you won't see it again

**Save this token somewhere safe** — you'll need it when pushing to GitHub for the first time.

*Note: Claude Desktop cannot access GitHub's authenticated settings pages.*

---

### Step 9: Install Docker Desktop

👤 **Human Required**

1. Download from [docker.com/products/docker-desktop](https://www.docker.com/products/docker-desktop/)
2. Open the `.dmg` file
3. Drag Docker to Applications
4. Launch Docker from Applications
5. **Approve the macOS permission dialogs** (this is why Claude can't do it)
6. Complete the setup wizard (you can skip signing in)
7. Wait for Docker to start (whale icon appears in menu bar)

Verify Docker:
```bash
docker --version
docker run hello-world
```

*Note: Claude Desktop cannot install .dmg files or approve macOS security dialogs.*

---

### Step 10: Create the Ralph Docker Image

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Create a Docker image for the Ralph Wiggum development sandbox. 

Create a directory at ~/ralph-docker and inside it create a Dockerfile with these specifications:
- Base image: ubuntu:24.04
- Install: curl, git, ca-certificates, gnupg
- Install Node.js 20.x from NodeSource
- Install globally via npm: @anthropic-ai/claude-code, serve, eslint, prettier
- Create a non-root user called "ralph"
- Set working directory to /workspace
- Configure git with name "Ralph Wiggum" and email "ralph@example.com"
- Set default branch to main

After creating the Dockerfile, build the image with the tag "ralph-sandbox". Show me the build output and confirm success.
```

**What this does:**
- Creates `~/ralph-docker/Dockerfile`
- Runs `docker build -t ralph-sandbox .`

---

### Step 11: Verify All Prerequisites

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Create a prerequisites verification script at ~/check-ralph-prereqs.sh that checks for:
- Homebrew installed
- Node.js installed (show version)
- npm installed (show version)
- Git installed (show version)
- Docker installed (show version)
- Claude CLI installed (show version)
- ANTHROPIC_API_KEY environment variable set (show first 12 chars only)
- Docker daemon running
- ralph-sandbox Docker image exists
- Git credential helper configured

Use colored output (green checkmarks for pass, red X for fail).
Make the script executable and run it. Show me the results.
```

**What this does:**
- Creates a verification script
- Runs it and shows status of all prerequisites

---

## Project Initialization

### Step 1: Create Project with Full Ralph Structure

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Create a new project for a Partner Compatibility Rating App at ~/Projects/compatibility-app with the complete Ralph Wiggum structure.

Create these directories:
- specs/
- src/
- src/lib/

Create these files with appropriate content:

1. README.md - Brief project description

2. src/index.html - Basic HTML5 boilerplate with:
   - Title: "Compatibility App"
   - Link to styles.css
   - A div with id="app" containing an h1 and loading message
   - Script tag for scripts.js

3. src/styles.css - Basic styles with:
   - Box-sizing border-box reset
   - System font stack
   - Light gray background (#F9FAFB)
   - Centered #app container with max-width 600px
   - Purple (#7C3AED) h1 color

4. src/scripts.js - Basic app scaffold with:
   - IIFE wrapper with 'use strict'
   - Console log on load
   - Empty state object for room, partnerId, ratings
   - Init function that runs on DOMContentLoaded

5. AGENTS.md - Operational guide containing:
   - Project overview (Partner Compatibility Rating App)
   - Build & run instructions (npx serve src -p 3000)
   - Validation commands (eslint, prettier)
   - Tech stack (vanilla HTML/CSS/JS, localStorage)
   - Project structure diagram
   - Codebase patterns (state management, DOM updates, naming conventions)
   - Placeholder for known issues

6. IMPLEMENTATION_PLAN.md - Empty file (Ralph will generate this)

7. loop.sh - The Ralph loop script that:
   - Parses arguments for plan/build mode and max iterations
   - Shows colored header with mode, prompt file, branch, iteration limit
   - Validates: prompt file exists, git repo initialized, claude CLI available, API key set
   - Main loop that: runs claude with -p --dangerously-skip-permissions --output-format=stream-json --model opus --verbose
   - Pushes to git after each iteration
   - Shows timing information
   - Make it executable

8. run-ralph.sh - Docker run helper that:
   - Checks Docker is running
   - Checks API key is set
   - Checks ralph-sandbox image exists
   - Mounts project directory to /workspace
   - Passes ANTHROPIC_API_KEY to container
   - Forwards port 3000
   - Supports: no args (interactive shell), "plan N" (planning), "build N" or just "N" (building)
   - Shows helpful usage instructions
   - Make it executable

9. PROMPT_plan.md - Planning mode prompt that instructs Claude to:
   - Study specs/* with parallel Sonnet subagents
   - Study existing IMPLEMENTATION_PLAN.md if present
   - Study src/lib/* for shared utilities
   - Compare specs against src/* code
   - Create/update IMPLEMENTATION_PLAN.md with prioritized tasks
   - Use Ultrathink for analysis
   - NOT implement anything, only plan
   - Include the ultimate goal: partner compatibility rating app with rooms, sliders, real-time sync, scores

10. PROMPT_build.md - Building mode prompt that instructs Claude to:
    - Study specs/* with parallel Sonnet subagents
    - Study IMPLEMENTATION_PLAN.md
    - Choose most important task
    - Search codebase before implementing (don't assume)
    - Use up to 500 Sonnet subagents for reads, 1 for builds
    - Use Opus subagents for complex reasoning
    - Run tests after implementing
    - Update IMPLEMENTATION_PLAN.md with discoveries
    - Git add, commit, and push when tests pass
    - Include numbered guardrails (99999, 999999, etc.) for:
      - Capturing the why in documentation
      - Single sources of truth
      - Creating git tags on success
      - Adding debug logging if needed
      - Keeping IMPLEMENTATION_PLAN.md current
      - Updating AGENTS.md with operational learnings (keep brief)
      - Documenting bugs even if unrelated
      - Implementing completely (no placeholders)
      - Cleaning completed items from plan
      - Fixing spec inconsistencies with Opus subagent
      - Keeping AGENTS.md operational only

Initialize a git repository and create the initial commit with message "Setup Ralph Wiggum project structure".

Show me the final directory structure and confirm all files were created.
```

**What this does:**
- Creates complete project structure
- Writes all configuration files
- Initializes git repository
- Creates initial commit

---

### Step 2: Create GitHub Repository

👤 **Human Required**

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** in the top right → **New repository**
3. Repository name: `compatibility-app`
4. Select **Private** (recommended)
5. Do NOT initialize with README (we already have one)
6. Click **Create repository**
7. Copy the repository URL (e.g., `https://github.com/YOUR-USERNAME/compatibility-app.git`)

*Note: Claude Desktop cannot create GitHub repositories without pre-authorized OAuth tokens.*

---

### Step 3: Connect Local Repo to GitHub

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop (replace YOUR-USERNAME):

```
Connect my local git repository at ~/Projects/compatibility-app to GitHub.

Add the remote origin: https://github.com/YOUR-USERNAME/compatibility-app.git

Push the main branch to GitHub with upstream tracking.

Note: If this is the first push, I may be prompted for GitHub credentials. My username is YOUR-USERNAME and I'll enter my Personal Access Token when prompted.

Show me the output of the push and confirm the remote is configured correctly with `git remote -v`.
```

**What this does:**
- Runs `git remote add origin ...`
- Runs `git push -u origin main`

*Note: On first push, you'll be prompted for credentials. Enter your GitHub username and Personal Access Token (not password).*

---

## Define Requirements

This phase creates the specification files that tell Ralph what to build.

### Understanding Subagents

When the prompts say "use 500 parallel Sonnet subagents," this instructs Claude about how to work:

- **Subagent** = Claude spawning a separate task/thread to do work
- **Sonnet** = A faster, cheaper Claude model (good for simple tasks)
- **Opus** = The most capable Claude model (good for complex reasoning)
- **Parallel** = Running multiple subagents at the same time

The numbers (250, 500) are guidance about parallelism—Claude decides how to actually execute this.

**Why different models?**
- Use **Sonnet subagents** for: reading files, searching code, simple updates
- Use **Opus subagents** for: complex decisions, debugging, architecture choices

This keeps costs down while maintaining quality where it matters.

---

### Create All Specification Files

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Create all specification files for the Partner Compatibility Rating App in ~/Projects/compatibility-app/specs/

Create these 5 spec files with detailed requirements and acceptance criteria:

1. specs/room-management.md
   Overview: Users create or join rooms using short codes
   Requirements for:
   - Room Creation: 6-char alphanumeric codes (exclude confusing chars like 0/O, 1/l), creator is "Partner 1", 24hr expiration
   - Room Joining: input field, validation, joiner is "Partner 2", max 2 users, error handling
   - Room State: localStorage persistence, unique partner IDs, handle refresh, "waiting for partner" state
   - Room Lifecycle: Active, Waiting, Expired, Complete states
   Acceptance criteria as checkboxes

2. specs/preference-system.md
   Overview: Categories of preferences with 1-10 sliders
   Requirements for:
   - 6 Categories with 5 items each:
     * Communication (conflict resolution, love language, check-ins, honesty style, venting vs solutions)
     * Lifestyle (early bird/night owl, tidiness, routine/spontaneity, introvert/extrovert, work-life)
     * Adventure (new foods, travel, risk tolerance, planning, exploration)
     * Finance (saving/spending, separate/joint, experiences/material, debt comfort, budgeting)
     * Family (children, extended family, roles, holidays, pets)
     * Intimacy (physical affection, quality time, emotional expression, reassurance, conversation depth)
   - Slider Interface: range 1-10, show value, endpoint labels, smooth animation, default to 5
   - Rating Flow: sequential categories, complete all before next, can go back, progress indicator, partner ratings hidden until both complete
   Acceptance criteria as checkboxes

3. specs/realtime-sync.md
   Overview: Both partners see updates in real-time
   Requirements for:
   - Sync Mechanism: poll-based (2 sec), localStorage for same-browser testing
   - Demo/Local Testing: storage events for cross-tab sync
   - Data to Sync: presence, category completion, ratings (after mutual completion), progress %
   - Connection Status: online/away indicator, 30 sec timeout
   - State Management: single source of truth, key structure room:{code}:state, timestamps, last-write-wins
   Acceptance criteria as checkboxes

4. specs/scoring.md
   Overview: Calculate and display compatibility scores
   Requirements for:
   - Scoring Algorithm:
     * Per-item: 10 - |rating1 - rating2| (max 10, min 1)
     * Per-category: average of items as percentage
     * Overall: average of categories as percentage
   - Score Display: percentage with decimal, color coding (green 80%+, yellow 50-79%, red <50%), bar chart
   - Item Comparison: side-by-side ratings, highlight matches (within 2), highlight mismatches (5+ apart)
   - Progressive Reveal: show category after both complete, update overall as you go
   Acceptance criteria as checkboxes

5. specs/user-interface.md
   Overview: Clean, mobile-first interface
   Requirements for:
   - Screen States: Landing (create/join), Waiting (code display, copy button), Rating (category, progress, slider, nav), Results (score, breakdown, details, share)
   - Layout: mobile-first 320px min, single column, centered 600px max, 44px touch targets
   - Visual Design: purple primary #7C3AED, green #10B981, amber #F59E0B, red #EF4444, gray background #F9FAFB
   - Animations: smooth transitions, slider animation, progress fills, score reveals
   - Accessibility: semantic HTML, ARIA labels, keyboard nav, focus indicators, color not sole indicator
   - Error States: inline messages, toast notifications, field highlighting, recovery actions
   Acceptance criteria as checkboxes

After creating all specs, commit them with message "Add requirement specifications" and push to GitHub.

Show me a summary of each spec file created.
```

**What this does:**
- Creates all 5 specification files with full detail
- Commits and pushes to GitHub

---

## Planning Mode

Now we use Ralph to generate an implementation plan.

### Step 1: Start Docker Desktop

👤 **Human Required**

1. Open **Docker Desktop** from your Applications folder
2. Wait for the whale icon to appear in your menu bar
3. Wait for it to say "Docker Desktop is running"

*Note: Claude Desktop cannot launch macOS applications.*

---

### Step 2: Enter Docker Sandbox and Run Planning

👤 **Human Required**

Open Terminal and run:

```bash
cd ~/Projects/compatibility-app
./run-ralph.sh
```

This opens an interactive shell inside the Docker container. Once inside, run:

```bash
./loop.sh plan 3
```

This runs up to 3 planning iterations. Ralph will:
1. Read all spec files in `specs/`
2. Analyze existing code in `src/`
3. Compare specs vs. code (gap analysis)
4. Generate `IMPLEMENTATION_PLAN.md` with prioritized tasks

**What "Ultrathink" means:** When Ralph sees this instruction, it engages in deeper, more thorough reasoning—considering more possibilities, checking its work, and thinking through edge cases. It's Claude-specific terminology that triggers more deliberate processing.

*Note: Claude Desktop cannot enter interactive Docker sessions or run the Ralph loop.*

---

### Step 3: Review the Plan

👤 **Human Required** (but Claude Desktop can help analyze)

After planning completes, review the generated plan. Inside the container or on your Mac:

```bash
cat IMPLEMENTATION_PLAN.md
```

**Good plan characteristics:**
- Logical ordering (foundations first)
- Tasks are appropriately sized (not too big)
- All spec requirements covered
- No duplicate tasks
- Clear task descriptions

If the plan looks wrong, regenerate it:

```bash
rm IMPLEMENTATION_PLAN.md
./loop.sh plan 3
```

---

## Building Mode

Now Ralph implements the application.

### Step 1: Run the Build Loop

👤 **Human Required**

Inside the Docker container:

```bash
# Run 20 iterations
./loop.sh 20

# Or unlimited (stop with Ctrl+C)
./loop.sh
```

*Note: Claude Desktop cannot run interactive Docker sessions.*

---

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
9. **Pushes** — Pushes to GitHub
10. **Clears** — Context window resets for next iteration

---

### Step 3: View Your App During Development

🤖 **Claude Desktop Can Help**

Copy and paste this prompt into Claude Desktop:

```
Start a local development server for my compatibility app at ~/Projects/compatibility-app

Run: npx serve src -p 3000

Tell me when it's running and remind me to open http://localhost:3000 in my browser.
```

**Alternative — open the file directly:**
```bash
open ~/Projects/compatibility-app/src/index.html
```

Note: Some features (like localStorage sync across tabs) may not work with `file://` URLs.

**Tip:** Keep a browser tab open and refresh after each Ralph iteration to see changes.

---

### Step 4: Monitor Progress

🤖 **Claude Desktop Can Help**

Copy and paste this prompt into Claude Desktop:

```
Show me the recent git activity for my compatibility app at ~/Projects/compatibility-app

1. Show the last 15 commits (oneline format)
2. Show the current contents of IMPLEMENTATION_PLAN.md
3. Count how many tasks are marked complete vs incomplete

Give me a summary of the project's progress.
```

---

### Step 5: Context Window Limits

Claude has a ~200K token context window. If a single iteration tries to:
- Read too many files at once
- Generate too much code
- Process too much information

...it may fail or produce poor results.

**Signs of context issues:**
- Ralph produces incomplete code
- Ralph "forgets" instructions partway through
- Responses get cut off

**Solutions:**
- Break large specs into smaller ones
- Split large files into smaller modules
- Add to PROMPT_build.md: "Focus on ONE small piece at a time"

---

### Step 6: Stopping and Resuming

👤 **Human Required**

**To stop safely:**
1. Wait for current iteration to commit (watch for "Iteration X complete")
2. Press `Ctrl+C`
3. Your progress is saved in git commits and IMPLEMENTATION_PLAN.md

**To resume later:**
```bash
# Enter the sandbox
./run-ralph.sh

# Continue building
./loop.sh 20
```

Ralph reads IMPLEMENTATION_PLAN.md each iteration, so it picks up where it left off.

**If you stopped mid-iteration (before commit):**
```bash
# Check for uncommitted changes
git status

# If changes look good, commit them manually
git add -A
git commit -m "Manual commit of partial work"

# If changes look broken, discard them
git reset --hard HEAD
```

---

## Deployment

### Step 1: Create Netlify Account

👤 **Human Required**

1. Go to [netlify.com](https://netlify.com)
2. Click **Sign up**
3. Choose **Sign up with GitHub** (easiest option)
4. Authorize Netlify to access your GitHub account
5. Complete any additional profile setup

*Note: Claude Desktop cannot create accounts or complete OAuth flows.*

---

### Step 2: Connect Repository to Netlify

👤 **Human Required**

1. In Netlify dashboard, click **Add new site** → **Import an existing project**
2. Click **GitHub**
3. If prompted, authorize Netlify to access your repositories
4. Find and select your `compatibility-app` repository
   - Private repositories appear here too!

*Note: Claude Desktop cannot complete OAuth authorization flows.*

---

### Step 3: Configure Build Settings

👤 **Human Required**

Netlify will ask for build settings. Enter:

| Field | What to Enter |
|-------|---------------|
| Branch to deploy | `main` |
| Base directory | *(leave blank)* |
| Build command | *(leave blank)* |
| Publish directory | `src` |

**Important:** Enter exactly `src` (not `/src` or `./src`).

Click **Deploy site**

---

### Step 4: Wait for First Deploy

👤 **Human Required**

Netlify builds your site in about 30 seconds. You'll see:
- Build log scrolling
- "Deploy published" when complete
- A random URL like `random-name-123456.netlify.app`

Click the URL to see your live site!

---

### Step 5: Automatic Deploys

From now on:
1. Ralph makes changes in Docker
2. Ralph commits and pushes to GitHub
3. Netlify automatically detects the push
4. Site updates live within 30-60 seconds

The `loop.sh` script already includes `git push` after each commit, so this happens automatically during Ralph iterations.

---

### Step 6: (Optional) Custom Domain

👤 **Human Required**

If you want `yourname.com` instead of `random.netlify.app`:

1. Buy a domain from [Namecheap](https://namecheap.com), [Google Domains](https://domains.google), or [Porkbun](https://porkbun.com) (~$10-15/year for .com)
2. In Netlify, go to **Site settings** → **Domain management**
3. Click **Add custom domain**
4. Enter your domain name
5. Follow Netlify's instructions to configure DNS

Netlify provides free SSL certificates automatically.

---

### Troubleshooting Netlify

**"Deploy failed: publish directory not found"**
- Make sure `src` folder exists and was pushed to GitHub
- Check that `src/index.html` exists
- Verify publish directory is exactly `src` (no slashes)

**Changes not appearing on live site**
- Check Netlify's **Deploys** tab for errors
- Verify `git push` succeeded
- Try triggering a manual deploy: **Deploys** → **Trigger deploy**

**Site shows old version**
- Hard refresh your browser: Cmd+Shift+R
- Clear browser cache
- Wait a minute for CDN propagation

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
| **Incomplete features** | Placeholders left behind | Add: "NEVER create placeholders. Implement completely or skip." |

---

### Adding Guardrails

🤖 **Claude Desktop Can Do This**

Copy and paste this prompt into Claude Desktop:

```
Add additional guardrails to my Ralph build prompt at ~/Projects/compatibility-app/PROMPT_build.md

Append these new numbered guardrails after the existing ones (continue the 9s pattern):

1. CRITICAL: Before implementing ANY feature, use at minimum 3 subagents to search for existing implementations. Report findings before writing code.

2. MANDATORY: Run 'npx eslint src/' before every commit. Fix all errors. No commits with lint errors.

3. ONE task per iteration. After completing ONE task, commit and exit. Do not continue to other tasks.

4. NEVER create placeholder or stub implementations. If you cannot fully implement a feature, document it in IMPLEMENTATION_PLAN.md and move to the next task.

Show me the updated guardrails section of the file.
```

---

### Regenerating the Plan

🤖 **Claude Desktop Can Help Prepare**

Copy and paste this prompt into Claude Desktop:

```
Delete the implementation plan for my compatibility app to prepare for regeneration:

rm ~/Projects/compatibility-app/IMPLEMENTATION_PLAN.md

Confirm the file was deleted. 

Remind me that I need to:
1. Enter the Docker sandbox: ./run-ralph.sh
2. Run planning mode: ./loop.sh plan 3
```

Then you (human) must run the planning loop in Docker.

---

### Escape Hatches

| Command | Purpose |
|---------|---------|
| `Ctrl+C` | Stop the loop immediately |
| `git reset --hard` | Revert all uncommitted changes |
| `git reset --hard HEAD~1` | Undo the last commit |
| `rm IMPLEMENTATION_PLAN.md` | Force fresh planning |
| `git stash` | Temporarily save uncommitted changes |
| `exit` | Leave Docker container |

---

## Troubleshooting

### Docker Issues

**"Cannot connect to Docker daemon"**

👤 **Human Required**
```bash
# Start Docker Desktop from Applications
open -a Docker

# Wait for whale icon to appear in menu bar, then verify
docker info
```

---

**"Image not found: ralph-sandbox"**

🤖 **Claude Desktop Can Do This**

```
Rebuild the Ralph Docker image.

Navigate to ~/ralph-docker and run docker build with tag ralph-sandbox.

Show me the build output and confirm success.
```

---

**"Permission denied" on project files**

🤖 **Claude Desktop Can Do This**

```
Fix file permissions for my compatibility app project.

Run: sudo chown -R $(whoami) ~/Projects/compatibility-app

Note: I may need to enter my password.

Confirm the permissions are fixed by listing the directory.
```

---

### Claude Issues

**"ANTHROPIC_API_KEY not set"**

🤖 **Claude Desktop Can Do This**

```
Check if my Anthropic API key is set in my shell configuration.

1. Check if ANTHROPIC_API_KEY exists in ~/.zshrc
2. Show me the current value of the environment variable (first 12 chars only)
3. If not set, remind me that I need to add it

Source the zshrc file to reload if needed.
```

---

**"Rate limited" or API errors**

Wait 60 seconds and retry. Check your usage at [console.anthropic.com](https://console.anthropic.com).

---

**"Insufficient credits" or payment errors**

👤 **Human Required**

1. Go to [console.anthropic.com](https://console.anthropic.com)
2. Check **Billing** section
3. Add payment method or credits

---

### Git Issues

**"Push rejected" or "Authentication failed"**

🤖 **Claude Desktop Can Help Diagnose**

```
Diagnose git remote issues for ~/Projects/compatibility-app

1. Check if remote origin is configured: git remote -v
2. Check current branch: git branch
3. Check if there are commits to push: git status

If the remote isn't configured, show me the command to add it (I'll need to fill in my GitHub username).

If authentication is the issue, remind me to use my Personal Access Token (not password) when prompted.
```

---

**"Merge conflicts"**

🤖 **Claude Desktop Can Do This**

```
Help me resolve git conflicts in ~/Projects/compatibility-app

1. Show me git status to see conflicted files
2. For each conflicted file, show me the conflict markers
3. Ask me which version I want to keep (theirs, mine, or let me see both)
4. After I decide, stage the resolved files and create a commit

Start by showing me the current status.
```

---

### Loop Issues

**Ralph keeps doing the same task**

🤖 **Claude Desktop Can Do This**

```
The Ralph loop seems stuck. Help me diagnose and fix it.

1. Show me the current IMPLEMENTATION_PLAN.md from ~/Projects/compatibility-app
2. Check if the "stuck" task is actually marked as complete
3. If not marked complete but code exists, update the plan to mark it done
4. If the plan looks corrupted, delete it (I'll regenerate in Docker)

Show me what you find.
```

---

**Ralph creates broken code**

🤖 **Claude Desktop Can Do This**

```
Ralph created broken code. Help me revert and prepare to retry.

In ~/Projects/compatibility-app:

1. Show me the last 5 commits
2. Revert to the commit before the broken one: git reset --hard HEAD~1
3. Force push to update GitHub: git push --force
4. Show me the current state of the code

Remind me to add more specific guardrails to PROMPT_build.md before retrying.
```

---

**Loop exits immediately**

🤖 **Claude Desktop Can Do This**

```
The Ralph loop exits immediately. Help me diagnose.

Check ~/Projects/compatibility-app:

1. Verify PROMPT_build.md exists and has content
2. Verify loop.sh exists and is executable
3. Check loop.sh for syntax errors: bash -n loop.sh
4. Verify we're in a git repository

Report what you find and suggest fixes.
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
# DOCKER COMMANDS (run from project directory)
# Human required - Claude Desktop cannot run Docker
# ═══════════════════════════════════════════

# Enter sandbox (interactive shell)
./run-ralph.sh

# Run planning directly (3 iterations)
./run-ralph.sh plan 3

# Run building directly (20 iterations)
./run-ralph.sh build 20
./run-ralph.sh 20              # shorthand

# ═══════════════════════════════════════════
# INSIDE DOCKER CONTAINER
# Human required - interactive session
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
# ON YOUR MAC (Claude Desktop can help)
# ═══════════════════════════════════════════

# View your app
npx serve src -p 3000       # Then open http://localhost:3000
open src/index.html         # Or open directly

# Monitor progress
git log --oneline -20
cat IMPLEMENTATION_PLAN.md

# ═══════════════════════════════════════════
# EMERGENCY COMMANDS (Claude Desktop can help)
# ═══════════════════════════════════════════

# Revert uncommitted changes
git reset --hard

# Undo last commit
git reset --hard HEAD~1

# Regenerate plan (delete then run planning in Docker)
rm IMPLEMENTATION_PLAN.md
```

### What Claude Desktop Can vs Cannot Do

| Task | Claude Desktop? | Why |
|------|-----------------|-----|
| Install packages via brew/npm | ✅ Yes | Shell commands |
| Create/edit project files | ✅ Yes | File operations |
| Run git commands | ✅ Yes | Shell commands |
| Configure shell/git settings | ✅ Yes | File/shell operations |
| Build Docker images | ✅ Yes | Shell commands |
| Create accounts (GitHub, Anthropic, Netlify) | ❌ No | CAPTCHA, email verification |
| Get API keys/tokens | ❌ No | Requires authenticated login |
| Authorize OAuth connections | ❌ No | Interactive browser flow |
| Install .dmg applications | ❌ No | macOS permission dialogs |
| Run interactive Docker sessions | ❌ No | Cannot control persistent terminal |
| Run the Ralph loop | ❌ No | Requires interactive Docker |

### Key Terminology

| Term | Meaning |
|------|---------|
| **JTBD** | Job to Be Done — the outcome users want |
| **Spec** | Specification file describing one feature area |
| **Subagent** | Claude spawning a task to work in parallel |
| **Sonnet** | Faster/cheaper Claude model for simple tasks |
| **Opus** | Most capable Claude model for complex reasoning |
| **Ultrathink** | Instruction for deeper, more thorough reasoning |
| **Backpressure** | Validation that rejects bad code before commit |
| **Context window** | ~200K tokens Claude can process at once |

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
│     🤖 Install: Node.js, Git, Claude CLI (via Claude)       │
│     👤 Create: Anthropic account, get API key               │
│     🤖 Configure: API key, Git identity (via Claude)        │
│     👤 Create: GitHub account, Personal Access Token        │
│     👤 Install: Docker Desktop                              │
│     🤖 Build: Docker image (via Claude)                     │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  2. PROJECT SETUP                                           │
│     🤖 Create: full project structure (via Claude)          │
│     👤 Create: GitHub repository                            │
│     🤖 Connect: local repo to GitHub (via Claude)           │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  3. DEFINE REQUIREMENTS                                     │
│     🤖 Write: all spec files (via Claude)                   │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  4. PLANNING MODE                                           │
│     👤 Start: Docker Desktop                                │
│     👤 Run: ./run-ralph.sh then ./loop.sh plan 3            │
│     👤 Review: IMPLEMENTATION_PLAN.md                       │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  5. BUILDING MODE                                           │
│     👤 Run: ./loop.sh 20 (in Docker)                        │
│     🤖 Monitor: progress via Claude                         │
│     🤖 Tune: add guardrails via Claude                      │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  6. DEPLOYMENT                                              │
│     👤 Create: Netlify account                              │
│     👤 Connect: GitHub repository                           │
│     👤 Configure: publish directory = src                   │
│     Auto-deploys on every push                              │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│  7. COMPLETE                                                │
│     App works, live at your-site.netlify.app                │
└─────────────────────────────────────────────────────────────┘
```

---

## Checklist

Use this checklist to track your progress:

```
□ Prerequisites
  □ 👤 Homebrew installed
  □ 🤖 Node.js installed (v18+)
  □ 🤖 Git installed
  □ 🤖 Claude Code CLI installed
  □ 👤 Anthropic account created
  □ 👤 API key obtained
  □ 🤖 API key set in shell
  □ 👤 Spending limit configured
  □ 🤖 Git identity configured
  □ 👤 GitHub account created
  □ 👤 Personal Access Token created
  □ 🤖 Git credential helper configured
  □ 👤 Docker Desktop installed and running
  □ 🤖 ralph-sandbox Docker image built
  □ 🤖 All prerequisites verified

□ Project Setup
  □ 🤖 Project structure created
  □ 🤖 Initial src files created
  □ 🤖 loop.sh and run-ralph.sh created
  □ 🤖 AGENTS.md and prompts created
  □ 👤 GitHub repository created
  □ 🤖 Local repo connected to GitHub

□ Requirements
  □ 🤖 All 5 spec files written
  □ 🤖 Specs committed and pushed

□ Planning
  □ 👤 Docker Desktop running
  □ 👤 Entered Docker sandbox
  □ 👤 Ran ./loop.sh plan 3
  □ 👤 Reviewed IMPLEMENTATION_PLAN.md

□ Building
  □ 👤 Ran ./loop.sh with iterations
  □ 🤖 Monitored progress
  □ 🤖 Added guardrails as needed

□ Deployment
  □ 👤 Netlify account created
  □ 👤 Repository connected to Netlify
  □ 👤 Build settings configured
  □ Site is live

□ Complete!
```

---

## All Claude Desktop Prompts (Copy-Paste Reference)

For convenience, here are all the Claude Desktop prompts from this tutorial in one place:

### Prerequisites Prompts

**Install Node.js and Git:**
```
Install Node.js and Git on my Mac using Homebrew. After installation, verify both are working by checking their versions. Show me the output of the version checks.
```

**Install Claude Code CLI:**
```
Install the Claude Code CLI globally using npm. After installation, verify it's working by checking the version. Show me the output.
```

**Set API Key (replace with your key):**
```
Add my Anthropic API key to my shell configuration so it persists across terminal sessions. My API key is: sk-ant-xxxxx (replace with actual key)

Add it to ~/.zshrc and reload the shell configuration. Then verify the key is set by echoing the first 15 characters (for security, don't show the full key).
```

**Configure Git Identity (replace with your info):**
```
Configure Git with my identity:
- Name: Your Full Name
- Email: your.email@example.com

Also enable the macOS Keychain credential helper so Git remembers my GitHub credentials. Show me the final git config to confirm.
```

**Build Docker Image:**
```
Create a Docker image for the Ralph Wiggum development sandbox. 

Create a directory at ~/ralph-docker and inside it create a Dockerfile with these specifications:
- Base image: ubuntu:24.04
- Install: curl, git, ca-certificates, gnupg
- Install Node.js 20.x from NodeSource
- Install globally via npm: @anthropic-ai/claude-code, serve, eslint, prettier
- Create a non-root user called "ralph"
- Set working directory to /workspace
- Configure git with name "Ralph Wiggum" and email "ralph@example.com"
- Set default branch to main

After creating the Dockerfile, build the image with the tag "ralph-sandbox". Show me the build output and confirm success.
```

**Verify Prerequisites:**
```
Create a prerequisites verification script at ~/check-ralph-prereqs.sh that checks for:
- Homebrew installed
- Node.js installed (show version)
- npm installed (show version)
- Git installed (show version)
- Docker installed (show version)
- Claude CLI installed (show version)
- ANTHROPIC_API_KEY environment variable set (show first 12 chars only)
- Docker daemon running
- ralph-sandbox Docker image exists
- Git credential helper configured

Use colored output (green checkmarks for pass, red X for fail).
Make the script executable and run it. Show me the results.
```

### Project Setup Prompt

**Create Full Project Structure:**
```
Create a new project for a Partner Compatibility Rating App at ~/Projects/compatibility-app with the complete Ralph Wiggum structure.

Create these directories:
- specs/
- src/
- src/lib/

Create these files with appropriate content:

1. README.md - Brief project description

2. src/index.html - Basic HTML5 boilerplate with:
   - Title: "Compatibility App"
   - Link to styles.css
   - A div with id="app" containing an h1 and loading message
   - Script tag for scripts.js

3. src/styles.css - Basic styles with:
   - Box-sizing border-box reset
   - System font stack
   - Light gray background (#F9FAFB)
   - Centered #app container with max-width 600px
   - Purple (#7C3AED) h1 color

4. src/scripts.js - Basic app scaffold with:
   - IIFE wrapper with 'use strict'
   - Console log on load
   - Empty state object for room, partnerId, ratings
   - Init function that runs on DOMContentLoaded

5. AGENTS.md - Operational guide containing:
   - Project overview (Partner Compatibility Rating App)
   - Build & run instructions (npx serve src -p 3000)
   - Validation commands (eslint, prettier)
   - Tech stack (vanilla HTML/CSS/JS, localStorage)
   - Project structure diagram
   - Codebase patterns (state management, DOM updates, naming conventions)
   - Placeholder for known issues

6. IMPLEMENTATION_PLAN.md - Empty file (Ralph will generate this)

7. loop.sh - The Ralph loop script that:
   - Parses arguments for plan/build mode and max iterations
   - Shows colored header with mode, prompt file, branch, iteration limit
   - Validates: prompt file exists, git repo initialized, claude CLI available, API key set
   - Main loop that: runs claude with -p --dangerously-skip-permissions --output-format=stream-json --model opus --verbose
   - Pushes to git after each iteration
   - Shows timing information
   - Make it executable

8. run-ralph.sh - Docker run helper that:
   - Checks Docker is running
   - Checks API key is set
   - Checks ralph-sandbox image exists
   - Mounts project directory to /workspace
   - Passes ANTHROPIC_API_KEY to container
   - Forwards port 3000
   - Supports: no args (interactive shell), "plan N" (planning), "build N" or just "N" (building)
   - Shows helpful usage instructions
   - Make it executable

9. PROMPT_plan.md - Planning mode prompt that instructs Claude to:
   - Study specs/* with parallel Sonnet subagents
   - Study existing IMPLEMENTATION_PLAN.md if present
   - Study src/lib/* for shared utilities
   - Compare specs against src/* code
   - Create/update IMPLEMENTATION_PLAN.md with prioritized tasks
   - Use Ultrathink for analysis
   - NOT implement anything, only plan
   - Include the ultimate goal: partner compatibility rating app with rooms, sliders, real-time sync, scores

10. PROMPT_build.md - Building mode prompt that instructs Claude to:
    - Study specs/* with parallel Sonnet subagents
    - Study IMPLEMENTATION_PLAN.md
    - Choose most important task
    - Search codebase before implementing (don't assume)
    - Use up to 500 Sonnet subagents for reads, 1 for builds
    - Use Opus subagents for complex reasoning
    - Run tests after implementing
    - Update IMPLEMENTATION_PLAN.md with discoveries
    - Git add, commit, and push when tests pass
    - Include numbered guardrails (99999, 999999, etc.) for:
      - Capturing the why in documentation
      - Single sources of truth
      - Creating git tags on success
      - Adding debug logging if needed
      - Keeping IMPLEMENTATION_PLAN.md current
      - Updating AGENTS.md with operational learnings (keep brief)
      - Documenting bugs even if unrelated
      - Implementing completely (no placeholders)
      - Cleaning completed items from plan
      - Fixing spec inconsistencies with Opus subagent
      - Keeping AGENTS.md operational only

Initialize a git repository and create the initial commit with message "Setup Ralph Wiggum project structure".

Show me the final directory structure and confirm all files were created.
```

**Connect to GitHub (replace YOUR-USERNAME):**
```
Connect my local git repository at ~/Projects/compatibility-app to GitHub.

Add the remote origin: https://github.com/YOUR-USERNAME/compatibility-app.git

Push the main branch to GitHub with upstream tracking.

Note: If this is the first push, I may be prompted for GitHub credentials. My username is YOUR-USERNAME and I'll enter my Personal Access Token when prompted.

Show me the output of the push and confirm the remote is configured correctly with `git remote -v`.
```

### Requirements Prompt

**Create All Spec Files:**
```
Create all specification files for the Partner Compatibility Rating App in ~/Projects/compatibility-app/specs/

Create these 5 spec files with detailed requirements and acceptance criteria:

1. specs/room-management.md
   Overview: Users create or join rooms using short codes
   Requirements for:
   - Room Creation: 6-char alphanumeric codes (exclude confusing chars like 0/O, 1/l), creator is "Partner 1", 24hr expiration
   - Room Joining: input field, validation, joiner is "Partner 2", max 2 users, error handling
   - Room State: localStorage persistence, unique partner IDs, handle refresh, "waiting for partner" state
   - Room Lifecycle: Active, Waiting, Expired, Complete states
   Acceptance criteria as checkboxes

2. specs/preference-system.md
   Overview: Categories of preferences with 1-10 sliders
   Requirements for:
   - 6 Categories with 5 items each:
     * Communication (conflict resolution, love language, check-ins, honesty style, venting vs solutions)
     * Lifestyle (early bird/night owl, tidiness, routine/spontaneity, introvert/extrovert, work-life)
     * Adventure (new foods, travel, risk tolerance, planning, exploration)
     * Finance (saving/spending, separate/joint, experiences/material, debt comfort, budgeting)
     * Family (children, extended family, roles, holidays, pets)
     * Intimacy (physical affection, quality time, emotional expression, reassurance, conversation depth)
   - Slider Interface: range 1-10, show value, endpoint labels, smooth animation, default to 5
   - Rating Flow: sequential categories, complete all before next, can go back, progress indicator, partner ratings hidden until both complete
   Acceptance criteria as checkboxes

3. specs/realtime-sync.md
   Overview: Both partners see updates in real-time
   Requirements for:
   - Sync Mechanism: poll-based (2 sec), localStorage for same-browser testing
   - Demo/Local Testing: storage events for cross-tab sync
   - Data to Sync: presence, category completion, ratings (after mutual completion), progress %
   - Connection Status: online/away indicator, 30 sec timeout
   - State Management: single source of truth, key structure room:{code}:state, timestamps, last-write-wins
   Acceptance criteria as checkboxes

4. specs/scoring.md
   Overview: Calculate and display compatibility scores
   Requirements for:
   - Scoring Algorithm:
     * Per-item: 10 - |rating1 - rating2| (max 10, min 1)
     * Per-category: average of items as percentage
     * Overall: average of categories as percentage
   - Score Display: percentage with decimal, color coding (green 80%+, yellow 50-79%, red <50%), bar chart
   - Item Comparison: side-by-side ratings, highlight matches (within 2), highlight mismatches (5+ apart)
   - Progressive Reveal: show category after both complete, update overall as you go
   Acceptance criteria as checkboxes

5. specs/user-interface.md
   Overview: Clean, mobile-first interface
   Requirements for:
   - Screen States: Landing (create/join), Waiting (code display, copy button), Rating (category, progress, slider, nav), Results (score, breakdown, details, share)
   - Layout: mobile-first 320px min, single column, centered 600px max, 44px touch targets
   - Visual Design: purple primary #7C3AED, green #10B981, amber #F59E0B, red #EF4444, gray background #F9FAFB
   - Animations: smooth transitions, slider animation, progress fills, score reveals
   - Accessibility: semantic HTML, ARIA labels, keyboard nav, focus indicators, color not sole indicator
   - Error States: inline messages, toast notifications, field highlighting, recovery actions
   Acceptance criteria as checkboxes

After creating all specs, commit them with message "Add requirement specifications" and push to GitHub.

Show me a summary of each spec file created.
```

### Monitoring Prompts

**Start Development Server:**
```
Start a local development server for my compatibility app at ~/Projects/compatibility-app

Run: npx serve src -p 3000

Tell me when it's running and remind me to open http://localhost:3000 in my browser.
```

**Check Progress:**
```
Show me the recent git activity for my compatibility app at ~/Projects/compatibility-app

1. Show the last 15 commits (oneline format)
2. Show the current contents of IMPLEMENTATION_PLAN.md
3. Count how many tasks are marked complete vs incomplete

Give me a summary of the project's progress.
```

**Add Guardrails:**
```
Add additional guardrails to my Ralph build prompt at ~/Projects/compatibility-app/PROMPT_build.md

Append these new numbered guardrails after the existing ones (continue the 9s pattern):

1. CRITICAL: Before implementing ANY feature, use at minimum 3 subagents to search for existing implementations. Report findings before writing code.

2. MANDATORY: Run 'npx eslint src/' before every commit. Fix all errors. No commits with lint errors.

3. ONE task per iteration. After completing ONE task, commit and exit. Do not continue to other tasks.

4. NEVER create placeholder or stub implementations. If you cannot fully implement a feature, document it in IMPLEMENTATION_PLAN.md and move to the next task.

Show me the updated guardrails section of the file.
```

**Prepare Plan Regeneration:**
```
Delete the implementation plan for my compatibility app to prepare for regeneration:

rm ~/Projects/compatibility-app/IMPLEMENTATION_PLAN.md

Confirm the file was deleted. 

Remind me that I need to:
1. Enter the Docker sandbox: ./run-ralph.sh
2. Run planning mode: ./loop.sh plan 3
```

### Troubleshooting Prompts

**Rebuild Docker Image:**
```
Rebuild the Ralph Docker image.

Navigate to ~/ralph-docker and run docker build with tag ralph-sandbox.

Show me the build output and confirm success.
```

**Fix File Permissions:**
```
Fix file permissions for my compatibility app project.

Run: sudo chown -R $(whoami) ~/Projects/compatibility-app

Note: I may need to enter my password.

Confirm the permissions are fixed by listing the directory.
```

**Check API Key:**
```
Check if my Anthropic API key is set in my shell configuration.

1. Check if ANTHROPIC_API_KEY exists in ~/.zshrc
2. Show me the current value of the environment variable (first 12 chars only)
3. If not set, remind me that I need to add it

Source the zshrc file to reload if needed.
```

**Diagnose Git Issues:**
```
Diagnose git remote issues for ~/Projects/compatibility-app

1. Check if remote origin is configured: git remote -v
2. Check current branch: git branch
3. Check if there are commits to push: git status

If the remote isn't configured, show me the command to add it (I'll need to fill in my GitHub username).

If authentication is the issue, remind me to use my Personal Access Token (not password) when prompted.
```

**Resolve Git Conflicts:**
```
Help me resolve git conflicts in ~/Projects/compatibility-app

1. Show me git status to see conflicted files
2. For each conflicted file, show me the conflict markers
3. Ask me which version I want to keep (theirs, mine, or let me see both)
4. After I decide, stage the resolved files and create a commit

Start by showing me the current status.
```

**Diagnose Stuck Loop:**
```
The Ralph loop seems stuck. Help me diagnose and fix it.

1. Show me the current IMPLEMENTATION_PLAN.md from ~/Projects/compatibility-app
2. Check if the "stuck" task is actually marked as complete
3. If not marked complete but code exists, update the plan to mark it done
4. If the plan looks corrupted, delete it (I'll regenerate in Docker)

Show me what you find.
```

**Revert Broken Code:**
```
Ralph created broken code. Help me revert and prepare to retry.

In ~/Projects/compatibility-app:

1. Show me the last 5 commits
2. Revert to the commit before the broken one: git reset --hard HEAD~1
3. Force push to update GitHub: git push --force
4. Show me the current state of the code

Remind me to add more specific guardrails to PROMPT_build.md before retrying.
```

**Diagnose Loop Exit:**
```
The Ralph loop exits immediately. Help me diagnose.

Check ~/Projects/compatibility-app:

1. Verify PROMPT_build.md exists and has content
2. Verify loop.sh exists and is executable
3. Check loop.sh for syntax errors: bash -n loop.sh
4. Verify we're in a git repository

Report what you find and suggest fixes.
```

---

## Next Steps

Once your app is working:

1. **Test thoroughly** — Run through all user flows manually
2. **Share the URL** — Let others try your compatibility app!
3. **Iterate** — Add new specs, regenerate plan, build more features
4. **Learn** — Study the code Ralph wrote to understand the patterns

The Ralph Wiggum technique scales to larger projects—just add more specs and let Ralph work through the plan. The key is clear specifications and trusting the iteration process.

Happy building! 🚀
