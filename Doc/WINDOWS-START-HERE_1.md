# Windows laptops: start here

One page, one path, everyone on it. Developer or ops, this is the same setup, and it stops at the
same place: WSL working, your Claude seat linked to your Arus email, and one repo cloned that you can
open a PR against. Whatever your work actually needs after that (a runtime, a database, product
repos) is a second step, per person, and it is one file.

Budget **about an hour**, most of it downloads and one reboot. Nothing here is hard, and nothing here
can break your laptop.

Read a step before you type it. Every command block in Part 2 is meant to be run one line at a time,
not pasted as a block: pasting several lines at once into Windows Terminal garbles the output and
makes a working step look broken.

## The mental model

Four things, and it helps to know which is which before you start.

| | What it is | Where it lives |
|---|---|---|
| **WSL** | Windows Subsystem for Linux. It runs a real Linux kernel inside Windows, sharing your files and network. Not a dual boot, not a full virtual machine you manage. | Windows feature |
| **Ubuntu** | The Linux distribution that runs on WSL. This is where your repos, git, node and Claude Code live. | inside WSL |
| **Windows Terminal** | The terminal window. It opens a tab straight into Ubuntu. This is the Mac team's Ghostty. | Windows app |
| **Claude Code** | Claude, running in the terminal, reading and writing your files, running commands, handling git. The actual tool. | inside Ubuntu |

So: Windows Terminal is the window, Ubuntu is what is listening, Claude Code is the colleague you
talk to inside it.

**Why WSL and not Claude Code on Windows directly.** Claude Code does run natively on Windows, and we
are deliberately not doing that. Everything we have written assumes a Linux shell: the installer, the
skills, the deck pipeline, and every command in every start-here doc. On WSL your machine behaves the
same as the team's Macs, so a fix that works for one person works for everyone. One environment to
support, not two.

## Before you start: what an owner does

Three things have to happen for you, none of which you can do yourself. Ask Elan or Joe, and it takes
them about two minutes.

| What | Why you notice it |
|---|---|
| **A Claude seat invited to your Arus email** | You get an invite email. Without it, signing in gives you a personal account with no team access |
| **A GitHub org invite** | Accepting it once means future repo grants apply immediately instead of arriving as separate per-repo invitations |
| **The repo grant** | `./grant-repo-access.sh --profile windows-starter --user <your-github-username>` |

Accept the two invites before Part 2. Both arrive by email, and the GitHub one also shows at
https://github.com/notifications.

Know your **GitHub username** before you ask, because the grant needs it, and it is often not what
you would guess from your name.

---

## Part 1: the Windows side

### 1. WSL, checking first whether you already have it

Open **PowerShell as Administrator** (right-click Start, then "Terminal (Admin)" or "Windows
PowerShell (Admin)").

Some laptops arrive with WSL already installed. Check before you install anything:

```powershell
wsl --update
wsl --list --verbose
```

Read the result and take one of these three paths.

| What you see | What to do |
|---|---|
| `Ubuntu` listed, VERSION column says `2` | You already have it. Skip to step 2 |
| `Ubuntu` listed, VERSION column says `1` | `wsl --set-version Ubuntu 2`, then continue |
| Nothing listed, or "no installed distributions" | Run `wsl --install`, then **reboot when it asks** |

If `wsl --install` answers **"A distribution with the supplied name already exists"**, that is not an
error you need to fix. It means Ubuntu is registered already. Run `wsl --list --verbose` and follow
the table above.

After a fresh install and reboot, Ubuntu opens by itself and asks you to create a **UNIX username and
password**.

- Use something short and lowercase for the username, for example `suriya`.
- **Remember that password.** It is your Linux `sudo` password, it has nothing to do with your
  Windows login, and there is no reset flow you will enjoy.

If Ubuntu opens straight to a prompt without asking for a username, that setup already happened.
Confirm with `whoami`, which should print your Linux username.

If `wsl --install` fails complaining about virtualization or the Virtual Machine Platform, it is a
BIOS setting (Intel VT-x or AMD-V) that IT or Joe can turn on. That is the one failure here that is
not fixable from Windows.

### 2. Windows Terminal, and make Ubuntu the default tab

Windows 11 has it already. On Windows 10, install **Windows Terminal** from the Microsoft Store.

Open it, click the dropdown arrow next to the `+`, and pick **Ubuntu**. Use this, not the old Command
Prompt window.

Then make Ubuntu the default, or every new tab you open with the `+` button lands in PowerShell and
none of the commands in this doc will work:

**Settings -> Startup -> Default profile -> Ubuntu**, then Save.

### 3. The font, and do it now rather than later

The shell prompt draws its git branch and folder icons with a patched font. Without it you get empty
boxes and rectangles, which looks like a broken install but is only a missing font.

Install it now, before Part 2, because the prompt setup wizard at the end of Part 2 asks you whether
various icons look right. Without the font those questions are unanswerable.

Install **MesloLGS NF** on **Windows** (not inside Ubuntu: WSL cannot install a font Windows uses).
The four files and the instructions are in the powerlevel10k font section:
https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k

Download all four, select them, right-click, **Install**. Then in Windows Terminal:
**Settings -> Ubuntu -> Appearance -> Font face -> MesloLGS NF**, and Save.

### 4. VS Code, if you want an editor

Install **VS Code on Windows**, then its **WSL** extension. Inside Ubuntu, `code .` in any repo opens
that folder properly, with the editor on Windows and the files staying on Linux.

Do not install VS Code inside Ubuntu, and do not open Linux files through a `C:\` path. The WSL
extension exists precisely so you never have to.

---

## Part 2: inside Ubuntu

Open the Ubuntu tab in Windows Terminal. Everything from here is typed there, **one line at a time**.

```bash
# 1. bring the distro up to date (asks for your Linux password)
sudo apt update && sudo apt upgrade -y
```

```bash
# 2. everything the installer needs, installed up front
sudo apt install -y git curl wget gh unzip zip jq zsh build-essential
```

Installing these yourself rather than leaving them to the installer is deliberate. `apt` installs a
list all or nothing, so one unavailable package fails the whole batch, and the installer's own list
contains one package that is not available on every Ubuntu release. Getting these in first means the
installer finds them present and moves on.

```bash
# 3. wslu, which lets a Linux command open your Windows browser. Optional
sudo apt install -y wslu
```

If that answers `Unable to locate package wslu`, your Ubuntu release does not have it in the archive
yet (Ubuntu 26.04 "Resolute" does not, as of August 2026). **Skip it and carry on.** The only
consequence is that the two sign-ins below print a URL instead of opening your browser, and you copy
that URL into your Windows browser by hand. Two logins, once, then never again.

```bash
# 4. sign in to GitHub
#    choose: GitHub.com -> HTTPS -> Yes (authenticate git) -> Login with a web browser
gh auth login
```

It shows a one-time code, then either opens your browser or prints
`https://github.com/login/device` for you to open yourself. Either way, paste the code there and
approve. It finishes with `Logged in as <your-github-username>`.

```bash
# 5. clone this repo into the fixed location every Windows machine here uses
mkdir -p ~/arus/code/arus-innovation
gh repo clone arus-innovation/arus-onboarding ~/arus/code/arus-innovation/arus-onboarding
```

```bash
# 6. run the installer
cd ~/arus/code/arus-innovation/arus-onboarding && ./install-wsl.sh --profile windows-starter
```

**If step 2 says `Unable to locate package gh`**, your Ubuntu is older than 22.10 and needs GitHub's
own package repo. Paste this block, then re-run step 2:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg >/dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list >/dev/null
sudo apt update
```

**If step 5 says the destination already exists and is not empty**, someone set this machine up
before you. Check what is there rather than deleting it:

```bash
cd ~/arus/code/arus-innovation/arus-onboarding && git status
```

If that prints `On branch main` and a clean tree, the repo is already cloned correctly. Skip to step
6.

### What the installer does

It asks for your name and Arus email (for git commits), then: apt base packages, node, Claude Code,
zsh with the powerlevel10k prompt, the shared Arus context and skills symlinked out of this repo,
`~/.secrets.zsh` with variable names and no values, your git identity, and the repo list from your
profile.

It finishes with a short list of things to do by hand, which is Part 5 below. Read that list: one
item on it is normal rather than a failure, the `huddle` skill, whose own installer asks which agents
to install for and cannot be answered by a script.

It is **safe to re-run at any point**, and re-running after a `git pull` is how you pick up updated
skills and conventions. Anything already done is skipped, anything it would replace is backed up
first, and it never deletes.

It takes a few minutes and goes quiet in places. Let it finish. You are done when it prints
`Setup complete for: Windows starter (WSL baseline)` followed by a numbered "Next, by hand" list.

### Confirm it landed, before you go further

Four one-line checks. Run them in the same terminal, and read each answer:

```bash
which zsh
```
Expect a path such as `/usr/bin/zsh`. Blank means zsh is not installed.

```bash
grep "^$(whoami):" /etc/passwd
```
Expect the line to **end in** `/usr/bin/zsh`. If it ends in a colon with nothing after it, zsh is
installed but is not your login shell.

```bash
ls ~/.oh-my-zsh/oh-my-zsh.sh
```
Expect the path back. `No such file or directory` means the oh-my-zsh framework is not in place.

```bash
claude --version
```
Expect a version number.

All four good, carry on. Any of them not good, the "A check in Part 2 failed" rows in the traps table
tell you exactly what to run. This mostly matters on a laptop that somebody part-configured before
you: a genuinely fresh machine normally passes all four.

### Then close the terminal and open a new one

**This step is not optional.** zsh and the new PATH only apply to a fresh shell.
`command not found: claude` almost always means this step was skipped.

Your new tab looks different: a coloured prompt with your username, the folder, and a clock on the
right. The first new tab also runs the **powerlevel10k configuration wizard**, which asks you whether
a series of shapes look right, then how you want the prompt to look.

Answer the shape questions honestly, by looking at your own screen. With the font installed in Part
1, step 3, they render properly and the answer is yes. Then, for the style questions, these are the
choices that suit our work, and none of them are permanent:

| Question | Pick | Why |
|---|---|---|
| Prompt Style | Classic or Rainbow | Taste. Both show folder and git branch clearly |
| Character Set | Unicode | You have the font, so use it |
| Prompt Height | **Two lines** | Your typing gets its own line, so long commands and Claude's output do not get cramped |
| Prompt Frame | No frame | Cleanest |
| Transient Prompt | **Yes** | Old commands collapse to a bare arrow, which keeps a long Claude session readable |
| Instant Prompt | Verbose | The recommended default |

If the wizard does not appear, or you want to redo it later, run `p10k configure`.

### Where your repos live

**Every Windows machine here uses the same path**, so nobody has to ask where somebody else keeps
their repos and every doc can print it literally:

```
~/arus/code/arus-innovation/arus-onboarding
~/arus/code/humini-ecosystem/<product>        once a product repo is granted to you
```

That is `~/arus/code`, one folder per org inside it. The installer defaults to it, your profile
states it, and the `hum` and `arus` aliases jump straight into the two org folders.

**Why it is not `C:\arus\code`.** That is the Windows filesystem, which WSL reaches as `/mnt/c`, and
working there costs you real things rather than theoretical ones: git and `npm install` are several
times slower because every file read crosses the filesystem boundary, and Linux file modes are not
preserved, so the executable bit on our own scripts is lost. The installer refuses a `/mnt` root
outright rather than letting someone discover this a week later.

**Windows still gets one fixed way in.** Two options, and the installer prints both with your real
paths filled in:

| | How | Needs |
|---|---|---|
| **Explorer path** | `\\wsl.localhost\Ubuntu\home\<you>\arus\code` in the address bar, or `explorer.exe .` from inside any repo | nothing |
| **A real `C:\arus\code`** | A symlink pointing at that same UNC path, so the folder exists on `C:` while the files stay on Linux | PowerShell as Administrator, once |

Read, browse and drag files through either. What you must not do is **copy** repos onto `C:` and work
on the copy: it diverges from the real one silently, which is a worse problem than a slow `git
status`.

Being straight about what has been tested: the WSL and Ubuntu side of this has been run end to end,
the Windows symlink has not been tried on one of our laptops yet. If it is refused on yours, pin the
UNC path to Quick Access instead, and say so, because it is one line in this doc either way.

---

## Part 3: linking your Claude seat

```bash
claude
```

First run asks which login method. Pick **1, Claude account with subscription**, not the Console or
third-party options. Our seats are subscription seats.

**A browser opens: sign in with your Arus email**, the same address the seat was invited to. Then it
asks whether you trust the files in the folder, which is yes for our own repos and is asked once per
folder.

The welcome banner is your confirmation. It should read:

```
Claude Team - Arus Innovation Pte Ltd
```

If it names a personal account instead, `/logout` then `/login` and pick the Arus address. `/status`
shows the same information at any time.

The seat is yours. Do not share a login: seats are per person, and the whole point of one per person
is that your context, your history and your `CLAUDE.md` are yours.

Then talk to it normally. No special syntax:

```
what is in this folder?
explain what this repo is for
what are the house rules I am expected to follow?
```

**Esc** interrupts it mid-answer, and you should use that early and often. **`/exit`** quits.
**`/help`** lists everything.

One thing worth knowing on day one: once Claude is running, the box at the bottom is a **chat box,
not a shell**. Typing `cd somewhere` there is a sentence to Claude, not a command your terminal runs,
and it will politely tell you so. To run shell commands yourself, `/exit` first.

New to the terminal? Read `docs/WELCOME-BEGINNER.md` in this repo next. It is written for a Mac, so
ignore the Ghostty and Homebrew lines, and everything else applies exactly.

---

## Part 4: the repo, and your first PR

You have one repo, and it is this one. That is on purpose: it is the repo that just set up your
machine, it carries the rules you are about to be held to, and a mistake in it costs nothing.

```bash
cd ~/arus/code/arus-innovation/arus-onboarding
claude
```

Check the banner names this folder and not `/home/<you>`. If it names your home folder, you started
Claude before changing directory: `/exit`, run the two lines again, in order.

A real first task, which you actually need anyway: give yourself your own profile.

```
copy profiles/windows-starter.env to profiles/<myname>.env, set my name and email,
and explain what each setting does. Then ship it.
```

Two things to watch while it works, both of which are the point of doing this task at all:

**`GIT_NAME` is your display name, not your GitHub username.** It is the name that appears on every
commit you ever make here. Other profiles read "Sundaran V" and "Vidhya Jayakumar", so yours should
read the same way. If the diff shows your GitHub username in `GIT_NAME`, say so and have it fixed
before you approve.

**Read the diff before you approve it.** Claude shows you the file it is about to write and the
commands it is about to run. That prompt is the point. Approving without reading is the habit this
first task exists to prevent.

**"ship it"** runs the `ship` skill: branch, commit, push the branch, open a PR, merge. You do not
need to memorise the git commands. When it finishes it prints the PR number and URL, and
`profiles/<myname>.env` is on `main`.

Product repos come after this. They are one line in your new profile plus one grant from an owner,
which is the "getting more access" section below.

---

## Part 5: the by-hand list, and you are done

The installer printed these. None of them are failures.

**1. Install the `huddle` skill.** Its own installer asks which agent to install for, which a script
cannot answer:

```bash
npx skills add muthuishere-agent-skills/huddle
```

Pick **Claude Code** when asked. If it reports the skill is already installed and identical, that is
a complete answer, skip it.

**2. Fill in your `~/.claude/CLAUDE.md`.** Easiest from inside Claude, in this repo:

```
open ~/.claude/CLAUDE.md and help me fill in the "Who I am" section
```

The more specific that file is, the less Claude guesses. Grow it over time by saying "add that to my
CLAUDE.md" whenever you explain the same thing twice.

**3. Optional, a `C:\arus\code` shortcut.** PowerShell as Administrator, once. The installer prints
this with your real path filled in. If Windows refuses it, pin the UNC path to Quick Access instead
and carry on, per "Where your repos live" above.

**4. Optional, a shortcut for starting work.** Two lines every morning is fine, but if you would
rather type one word, add an alias:

```bash
echo "alias arusgo='cd ~/arus/code/arus-innovation/arus-onboarding && claude'" >> ~/.zshrc
```

Open a new terminal, then `arusgo` starts Claude in the repo. An alias is better than putting `cd`
and `claude` directly in `~/.zshrc`, because that variant launches Claude in **every** terminal tab
you ever open, including the ones where you only wanted to run a git command.

That is the whole setup. You now have WSL working, a Claude seat on the Arus organisation, one repo,
and a merged PR with your name on it.

---

## The rules that apply to you

From `shared/conventions/ARUS-CONVENTIONS.md`, which is now loaded into every Claude session on your
machine.

**1. Never push directly to `main`.** Branch, commit, push the branch, open a PR, merge the PR.
Everyone follows this including the founders, including when someone literally says "push to main".
Branch naming is `<product>-<name>`.

This one needs your attention rather than your trust, because **nothing stops you mechanically.** Our
repos are private on a plan without branch protection, so a push to `main` would simply work. The
rule holds because people follow it.

**2. Spec first, and it is a hard rule.** Brainstorm, then logged decisions, then spec, then code. If
you are about to implement something and there is no spec, write the spec first.

**3. No em-dash characters.** Anywhere: docs, specs, commit messages, PR text, code comments. A
comma, colon, or parentheses instead.

**4. Secrets never travel.** No key, token, or password in a repo, a screenshot, or a chat message.
`~/.secrets.zsh` on your machine holds variable names and no values. Anything ever exposed gets
rotated, not just deleted.

---

## The WSL traps

Read the first row now, the rest when something looks wrong.

| Symptom or situation | What is going on |
|---|---|
| **Where to keep repos** | `~/arus/code`, the same on every Windows machine here, which is where the installer puts them. Never under `/mnt/c`, and never a copy on `C:`. See "Where your repos live" above |
| `wsl --install` says a distribution already exists | Not an error. Ubuntu is registered already. Run `wsl --list --verbose` and follow the table in Part 1, step 1 |
| A new tab opens PowerShell instead of Ubuntu | The default profile is still PowerShell. Windows Terminal **Settings -> Startup -> Default profile -> Ubuntu**. Meanwhile, the dropdown arrow next to `+` opens Ubuntu |
| `command not found: claude` | Close the terminal, open a new one. New tools need a fresh shell. The single most common first-morning issue |
| A check in Part 2 failed: `which zsh` is blank | `sudo apt install -y zsh`, then set it as your login shell with the next row |
| A check in Part 2 failed: your `/etc/passwd` line does not end in `/usr/bin/zsh` | `sudo chsh -s "$(which zsh)" "$(whoami)"`, then close the terminal and open a new one |
| A check in Part 2 failed: `~/.oh-my-zsh/oh-my-zsh.sh` is missing | The framework is absent while your theme folder survives. Restore the framework without touching `custom/`: `git clone --depth=1 https://github.com/ohmyzsh/ohmyzsh.git /tmp/ohmyzsh` then `find /tmp/ohmyzsh -maxdepth 1 -mindepth 1 ! -name custom -exec cp -r {} ~/.oh-my-zsh/ \;` then `rm -rf /tmp/ohmyzsh` |
| Prompt is still a plain `$` after the installer and a new terminal | zsh is not your login shell. The two rows above, in order |
| `Unable to locate package wslu` | Not available on your Ubuntu release. Skip it. Sign-in URLs get copied into your browser by hand instead, twice, ever |
| `apt install` fails and installs none of the list | `apt` is all or nothing. Install the list without the one package it cannot find, then deal with that one separately |
| Prompt shows empty boxes instead of icons | The MesloLGS NF font is not installed on Windows, or not selected in Windows Terminal. Part 1, step 3 |
| Typed `cd ...` or `claude` into Claude's own prompt box | That box is a chat box, not a shell. `/exit` first, then run shell commands yourself |
| Pasted several commands at once and the output looks garbled | Windows Terminal ran them together. Run them one line at a time, pressing Enter after each |
| Reaching your Linux files from Windows | Explorer address bar: `\\wsl.localhost\Ubuntu\home\<you>\arus\code` (older builds: `\\wsl$\Ubuntu\...`). Read and drag files freely. Just do not point a Windows Git or a Windows editor at a Linux repo |
| `sudo` asks for a password you do not know | It is the **Linux** password you created when Ubuntu first started, not your Windows login |
| Whole files show as changed in `git diff` | A Windows tool rewrote the line endings to CRLF. Edit Linux files with VS Code plus the WSL extension, or from inside Ubuntu, and this never happens |
| `npm install -g` fails with EACCES | The installer moves npm's global prefix to `~/.npm-global` so no global install needs sudo. If you hit this, you are in a shell that predates the change: open a new terminal. Do not fix it with `sudo npm` |
| Certificate or TLS errors after the laptop sleeps | The WSL clock drifted. `sudo hwclock -s` resets it |
| Ubuntu feels slow, or eats memory | WSL2 grows its memory as needed and does not always hand it back. Create `C:\Users\<you>\.wslconfig` with `[wsl2]` and `memory=8GB`, then `wsl --shutdown` in PowerShell. Also exclude your WSL folders from third-party antivirus, which is a common cause of slow git |
| A repo will not clone | That is a grant that has not happened yet, not something you broke. Check your email for GitHub invites, accept them, then re-run with `--repos-only` |
| Claude ignores something in your `CLAUDE.md` | `/context` shows what actually loaded this session |
| Claude is confidently wrong | It happens. Say so plainly, it corrects and continues. Verify anything factual that matters |
| Anything else | Run `claude`, describe what you are seeing, paste the error. Fastest path, every time |

---

## Getting more access later

Your profile is data, not code: `profiles/<name>.env`. A repo you need, a runtime you actually use,
write access somewhere new: all of it is one file and a PR, and none of it needs a script change.

```bash
# an owner runs this after your profile lists the repos
./grant-repo-access.sh --profile <name> --user <your-github-username>

# then, on your machine
cd ~/arus/code/arus-innovation/arus-onboarding && git pull && ./install-wsl.sh --profile <name> --repos-only
```

Access is explicit, per repo, per person. Both orgs run base permission `none`, so being in the org
grants nothing by itself. Write access is for the repos you own, read for the ones you reference. If
Claude hits a permission error pushing somewhere, that is the model working, not a broken setup.

## Keeping the machine current

```bash
cd ~/arus/code/arus-innovation/arus-onboarding && git pull && ./install-wsl.sh --profile <your-profile>
```

Skills and shared context are symlinks into this repo, so a `git pull` updates them on every machine
at once. `--skills-only` refreshes just those, `--repos-only` clones anything newly granted, and both
skip the slow package steps.

## Deliberately not installed

- **MCP servers and plugins.** They need interactive auth per machine and you should pick your own.
  `/plugin` and `/mcp` inside Claude.
- **Any product toolchain.** No JDK, no database, no Docker on the starter profile. Whoever needs one
  gets it named in their own profile rather than every laptop carrying it.
- **Real credentials.** `~/.secrets.zsh` has variable names and no values. There is no agreed source
  of truth for Arus credentials yet, so ask Elan or Joe rather than copying a key out of a chat
  message.
- **Permission-prompt skipping.** No profile enables `--dangerously-skip-permissions`. Those prompts
  are your steering wheel while you are learning what Claude is about to do.

---

## What has actually been tested

Run end to end on 25 August 2026, on a Windows 11 laptop, Ubuntu 26.04 LTS "Resolute Raccoon" on
WSL2, Claude Code 2.1.241, finishing with a merged PR. Everything in Parts 1 through 5 came out of
that run.

Two things in this doc have **not** been run on one of our laptops:

- The `C:\arus\code` symlink, which is optional and has a stated fallback.
- A first install on Ubuntu 24.04 or older. The `gh` package-repo block in Part 2 is carried over
  from the previous version of this doc and is unverified here.

If you hit something this page does not cover, add the row. One line in the traps table is worth more
than the next person rediscovering it.
