# Windows laptops: start here

One page, one path, everyone on it. Developer or ops, this is the same setup, and it stops at the
same place: WSL working, your Claude seat linked to your Arus email, and one repo cloned that you can
open a PR against. Whatever your work actually needs after that (a runtime, a database, product
repos) is a second step, per person, and it is one file.

**This page assumes you have never used a terminal.** Every command says what it does, and every step
says what you should see on screen when it worked. If your screen does not match, that is what "The
WSL traps" at the bottom is for.

Budget **about an hour**, most of it downloads and one reboot. Nothing here is hard, and nothing here
can break your laptop.

---

## Your progress checklist

Tick these off as you go. If you have to stop halfway, this tells you where to pick up.

```
[ ]  0. Owner has sent you two invites, and you accepted them
[ ]  1. Windows is new enough
[ ]  2. WSL and Ubuntu installed, Linux username and password created
[ ]  3. Windows Terminal opens Ubuntu by default
[ ]  4. MesloLGS NF font installed and selected
[ ]  5. Ubuntu updated, base packages installed
[ ]  6. Signed in to GitHub from Ubuntu
[ ]  7. arus-onboarding cloned
[ ]  8. Installer finished, four checks pass
[ ]  9. New terminal opens with the coloured prompt
[ ] 10. Claude signed in, banner says Arus Innovation Pte Ltd
[ ] 11. Your own profile shipped as a merged PR
[ ] 12. The by-hand list done
```

---

## How to read this page

**Grey boxes are things you type.** Like this:

```bash
whoami
```

Type or paste it, then press **Enter**. That is what "run this" means everywhere below.

**One line at a time.** Where a box has several lines, do them one by one, pressing Enter after each
and waiting for it to finish. Pasting a whole block at once makes Windows Terminal jumble the output,
and a step that worked fine will look like it broke.

**To paste into the terminal**, use **Ctrl+Shift+V**, or just right-click. Ordinary Ctrl+V also works
in most cases, but Ctrl+Shift+V always does.

**"You should see"** blocks show you what success looks like. Compare yours against them before
moving on. Getting a slightly different version number is fine. Getting an error is not.

**Nothing appears when you type a password.** No dots, no stars, the cursor does not move. That is
normal on Linux, not a frozen screen. Type it and press Enter.

**To scroll back** through output that went past too fast, use your mouse wheel.

---

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

---

## Step 0: what an owner does for you

Three things have to happen for you, none of which you can do yourself. Ask Elan or Joe, and it takes
them about two minutes.

| What | Why you notice it |
|---|---|
| **A Claude seat invited to your Arus email** | You get an invite email. Without it, signing in gives you a personal account with no team access |
| **A GitHub org invite** | Accepting it once means future repo grants apply immediately instead of arriving as separate per-repo invitations |
| **The repo grant** | `./grant-repo-access.sh --profile windows-starter --user <your-github-username>` |

**Accept both invites before Part 2.** They arrive by email, and the GitHub one also shows at
https://github.com/notifications.

**Two things to have ready before you ask:**

- **Your GitHub username.** Not your name, not your email. It is often not what you would guess. If
  you are not sure, sign in at https://github.com and click your profile picture, top right. The
  username is under your name.
- **Local administrator rights on the laptop.** Part 1 needs them. If your laptop is IT-managed and
  the "Administrator" option below is greyed out, that is a two-minute request to IT, not something
  you can work around.

---

## Part 1: the Windows side

About 25 minutes, including one reboot.

### 1.1 Check Windows is new enough

Hold the **Windows key** and press **R**. A small "Run" box opens. Type:

```
winver
```

Press Enter. A window appears with your Windows version.

**You should see** either:

- **Windows 11**, any version. You are fine.
- **Windows 10, Version 2004 or higher** (the build number, in brackets, is 19041 or higher). You are
  fine.

If it says Windows 10 with a version below 2004, run Windows Update first and come back. The
one-command WSL install below needs at least that.

Close the winver window.

### 1.2 Open PowerShell as Administrator, and pick the right one

**The reliable way:** right-click the **Start button** (the Windows logo, bottom left). In the menu
that opens, click **Terminal (Admin)** on Windows 11, or **Windows PowerShell (Admin)** on Windows
10. Click **Yes** when Windows asks for permission.

**You should see** a dark window whose title bar starts with the word **Administrator**. If the title
bar does not say Administrator, close it and open it again through the right-click menu. Half the
commands in this section fail silently without it.

**About the "(x86)" one.** If you search the Start menu for "powershell" instead of using the
right-click menu, you may see two results: **Windows PowerShell** and **Windows PowerShell (x86)**.
The (x86) one is the older 32-bit version. Prefer the one **without** (x86).

That said, if you have already opened the (x86) one, do not start over. The `wsl` commands below were
run from the (x86) window during testing and worked normally. It is a preference, not a blocker.

### 1.3 Check whether WSL is already installed

Some laptops arrive with WSL already on them. Check before installing anything. In the Administrator
PowerShell window, run these two, one at a time:

```powershell
wsl --update
```

```powershell
wsl --list --verbose
```

Now read what came back and pick your row:

| What you see | What it means | What to do |
|---|---|---|
| A list with `Ubuntu` and `2` in the VERSION column | Already installed and on the right version | **Skip to 1.5** |
| A list with `Ubuntu` and `1` in the VERSION column | Installed but on the old version | Run `wsl --set-version Ubuntu 2`, wait for it to finish, then **skip to 1.5** |
| "no installed distributions", or an error saying WSL is not installed | Nothing there yet | **Continue to 1.4** |

**One message that looks scary and is not:**

```
A distribution with the supplied name already exists.
Error code: Wsl/InstallDistro/ERROR_ALREADY_EXISTS
```

That means Ubuntu is already registered. It is not a failure and nothing is broken. Run
`wsl --list --verbose` and use the table above.

### 1.4 Install WSL and Ubuntu

Only if 1.3 told you to. In the same Administrator PowerShell window:

```powershell
wsl --install
```

This turns on the Windows features, installs WSL2, and downloads Ubuntu. It takes several minutes and
prints progress as it goes.

**Reboot when it asks you to.** The install is not finished until you do.

**After the reboot**, an Ubuntu window opens on its own and says something like
`Installing, this may take a few minutes...`. Then it asks you to create a username. The exact
wording can vary by WSL version, for example `Create a default/new UNIX user account:` or
`Enter new UNIX username:`. Either way, it wants the same thing:

- Type something **short and lowercase**, for example `suriya` or `priya`. No spaces, no capitals.
- Press Enter. It then asks for a password, twice.
- **Remember this password.** It is your Linux password, used whenever a command starts with `sudo`.
  It has **nothing to do with your Windows login**, and there is no pleasant way to reset it.
- Remember: nothing appears on screen while you type it. That is normal.

**You should see**, once it finishes, a prompt sitting there waiting, something like:

```
priya@LAPTOP-NAME:~$
```

If Ubuntu opens straight to a prompt and never asks for a username, that setup already happened on
this laptop. Confirm by running:

```bash
whoami
```

It should print a username. That is the one you already have.

If all good, then close and reopen the terminal.

### 1.5 Confirm you are on WSL version 2

Back in the Administrator PowerShell window:

```powershell
wsl --list --verbose
```

**You should see** something like:

```
  NAME      STATE           VERSION
* Ubuntu    Running         2
```

The VERSION column **must** say `2`. If it says `1`, run `wsl --set-version Ubuntu 2` and check
again.

If `wsl --install` failed complaining about virtualization or the Virtual Machine Platform, that is a
BIOS setting (Intel VT-x or AMD-V) that IT or Joe can turn on. It is the one failure here that is not
fixable from Windows.

You can close the PowerShell window now. You will not need Administrator again unless you want the
optional shortcut in Part 5.

### 1.6 Windows Terminal, and make Ubuntu the default tab

Windows 11 already has Windows Terminal. On Windows 10, install **Windows Terminal** from the
**Microsoft Store** (search the Start menu for "Microsoft Store", then search the store for "Windows
Terminal", then Get).

Open Windows Terminal. It probably opens a PowerShell tab. To get an Ubuntu tab, click the small
**dropdown arrow** next to the `+` at the top, and choose **Ubuntu**.

Now make Ubuntu the default, so you never have to think about it again:

1. Click the dropdown arrow next to `+`
2. Click **Settings**
3. On the **Startup** page, find **Default profile**
4. Change it to **Ubuntu**
5. Click **Save**

**Why this matters:** without it, clicking the `+` button opens a PowerShell tab, and every Ubuntu
command in this document will fail there with confusing errors. This catches people out constantly.

**You should see**, after saving, that a brand new tab opens straight into Ubuntu with a prompt like
`priya@LAPTOP-NAME:~$`.

### 1.7 Install the font, and do it now

The prompt you will end up with draws small icons for things like your git branch. Those icons need a
special font. Without it you get empty rectangles and it looks like a broken install when it is only
a missing font.

**Do this before Part 2**, because the prompt setup at the end of Part 2 literally asks you "does
this look like a diamond?" and you cannot answer that without the font.

1. Open this page in your browser:
   https://github.com/romkatv/powerlevel10k#meslo-nerd-font-patched-for-powerlevel10k
2. It lists **four** font files: Regular, Bold, Italic, Bold Italic. Download all four.
3. Open your Downloads folder, select all four files, right-click, choose **Install** (on Windows 11
   you may need "Show more options" first).
4. Back in Windows Terminal: dropdown arrow next to `+`, then **Settings**, then **Ubuntu** in the
   left sidebar, then **Appearance**, then set **Font face** to **MesloLGS NF**, then **Save**.

**You should see** the font name accepted in the dropdown. Nothing visibly changes yet, which is
fine. It matters later.

### 1.8 VS Code, if you want an editor

Optional, and you can come back to it.

Install **VS Code on Windows**, then its **WSL** extension. After that, typing `code .` inside any
repo in Ubuntu opens that folder properly, with the editor running on Windows and the files staying
on Linux.

Do not install VS Code inside Ubuntu, and do not open Linux files through a `C:\` path. The WSL
extension exists precisely so you never have to.

---

## Part 2: inside Ubuntu

About 20 minutes. Open the Ubuntu tab in Windows Terminal. Everything from here is typed there, **one
line at a time**.

### 2.1 Update Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
```

`sudo` means "run this as administrator", so it asks for the **Linux** password you created in 1.4.
Nothing appears while you type it.

This downloads and installs updates. On a fresh machine it can take a few minutes and scrolls a lot
of text. If it stops on a purple full-screen box asking about restarting services, press **Enter** to
accept the default.

**You should see** it finish and return you to your prompt, with a summary line near the end. Wording
varies by release, but nothing should say `E:` or `Error`.

### 2.2 Install everything the installer will need

```bash
sudo apt install -y git curl wget gh unzip zip jq zsh build-essential
```

Plain English: this installs git (version control), curl and wget (downloaders), gh (the GitHub
command-line tool), unzip and zip (archives), jq (reads JSON), zsh (the shell you will end up using),
and build-essential (compilers that other tools need).

**Why we install these ourselves** instead of letting the installer do it: `apt` installs a list all
or nothing, so if a single package on a list is unavailable, **the entire list fails and nothing gets
installed**. The installer's own list contains one package that is not available on every Ubuntu
release. Doing it here, without that package, means the installer finds everything already present
and moves straight past it.

**You should see** either packages downloading and `Setting up ...` lines, or
`... is already the newest version`. Both are success.

### 2.3 Install wslu, which is optional

```bash
sudo apt install -y wslu
```

`wslu` lets a Linux command open your Windows browser, which makes the two sign-ins below smoother.

**If it says `Unable to locate package wslu`, that is expected on newer Ubuntu and you should just
carry on.** Ubuntu 26.04 "Resolute Raccoon" does not have it in the archive as of August 2026. The
only consequence: the two sign-ins below print a web address instead of opening your browser, and you
copy that address into your browser yourself. Twice, ever. It is not worth solving.

### 2.4 Sign in to GitHub

```bash
gh auth login
```

It asks you a series of questions. Use the arrow keys to choose, then Enter:

| Question | Answer |
|---|---|
| What account do you want to log into? | **GitHub.com** |
| What is your preferred protocol for Git operations? | **HTTPS** |
| Authenticate Git with your GitHub credentials? | **Yes** |
| How would you like to authenticate GitHub CLI? | **Login with a web browser** |

It then shows a **one-time code** like `1E7B-C7CB`. Copy it. Then either your browser opens on its
own, or you get a line like:

```
! Failed opening a web browser at https://github.com/login/device
  Please try entering the URL in your browser manually
```

That message is fine, and it is what happens when 2.3 could not install `wslu`. Open
https://github.com/login/device in your normal Windows browser yourself, paste the code, and approve.

**You should see**, back in the terminal:

```
✓ Authentication complete.
✓ Logged in as your-github-username
```

If it says `You were already logged in to this account`, that is also success.

### 2.5 Clone the onboarding repo

Two lines, one at a time:

```bash
mkdir -p ~/arus/code/arus-innovation
```

```bash
gh repo clone arus-innovation/arus-onboarding ~/arus/code/arus-innovation/arus-onboarding
```

`mkdir -p` creates the folder. `gh repo clone` downloads the repo into it.

**You should see** `Cloning into ...` and a few progress lines.

**If it says the destination already exists and is not empty**, somebody set this laptop up before
you. Do not delete anything. Check what is there:

```bash
cd ~/arus/code/arus-innovation/arus-onboarding && git status
```

If that prints `On branch main` and `nothing to commit, working tree clean`, the repo is already
correctly cloned and you can move to 2.6.

**If it says `404` or `Could not resolve to a Repository`**, that is the repo grant not having
happened yet, not something you broke. Check your email and https://github.com/notifications for the
GitHub invite, accept it, and try again.

### 2.6 Run the installer

```bash
cd ~/arus/code/arus-innovation/arus-onboarding && ./install-wsl.sh --profile windows-starter
```

`cd` moves you into the folder. The `./install-wsl.sh` part runs the setup script.

It asks for **your full name** and **your Arus email**, which it uses for your git commits. Then it
works through fourteen numbered sections: base packages, node, Claude Code, the zsh prompt, the
shared Arus context and skills, `~/.secrets.zsh` (variable names, no values), your git identity, and
your repo list.

**It takes several minutes and goes quiet in places. Let it finish.**

**You should see**, at the end:

```
================================================
 Setup complete for: Windows starter (WSL baseline)
================================================
```

followed by a numbered **"Next, by hand"** list. That list is Part 5 of this document.

**One line in its output is normal rather than a failure:**

```
! huddle not installed (its installer needs an answer it could not get here)
```

The `huddle` skill's own installer asks which agent to install for, and a script cannot answer that.
You do it by hand in Part 5.

**Good to know:** this installer is safe to run again at any time. Re-running it after a `git pull`
is how you pick up updated skills and conventions. Anything already done is skipped, anything it
would replace is backed up first, and it never deletes.

### 2.7 Four checks before you go further

Run these one at a time and read each answer. They take ten seconds and save you an hour.

```bash
which zsh
```
**Expect** a path such as `/usr/bin/zsh`. Blank means zsh is not installed.

```bash
grep "^$(whoami):" /etc/passwd
```
**Expect** a line that **ends in** `/usr/bin/zsh`, like
`priya:x:1000:1000::/home/priya:/usr/bin/zsh`. If it ends in a colon with nothing after it, zsh is
installed but is not your login shell.

```bash
ls ~/.oh-my-zsh/oh-my-zsh.sh
```
**Expect** the path printed back. `No such file or directory` means the shell framework is not fully
in place.

```bash
claude --version
```
**Expect** a version number, such as `2.1.241`.

All four good, carry on to 2.8. Any of them not good, the rows in "The WSL traps" starting
"A check in Part 2 failed" tell you exactly what to run. On a genuinely fresh laptop all four
normally pass. They tend to matter only where somebody part-configured the machine before you.

### 2.8 Close the terminal, open a new one

**This is not optional and it is the single most common first-morning problem.** The new shell and
the new PATH only apply to a freshly opened terminal. `command not found: claude` almost always means
this step was skipped.

Close the whole Windows Terminal window, then open it again.

**You should see** a very different prompt: coloured, with your username and folder on the left and a
clock on the right, instead of a plain `$`.

### 2.9 Answer the prompt setup wizard

The first new terminal also runs the **Powerlevel10k configuration wizard** by itself. It asks you a
run of questions.

**First, several "does this look like a ..." questions** showing a shape: a diamond, a padlock, some
arrows. Look at your own screen and answer honestly with `y` or `n`. With the font installed back in
1.7, they render properly and the answer is `y`.

**Then, style questions.** None of these are permanent, and you can redo the whole thing later with
`p10k configure`. These are the choices that suit our work:

| Question | Pick | Why |
|---|---|---|
| Prompt Style | **Classic** or **Rainbow** | Taste. Both show folder and git branch clearly |
| Character Set | **Unicode** | You have the font, so use the nicer shapes |
| Prompt Height | **Two lines** | Your typing gets its own line, so long commands and Claude's output do not get cramped |
| Prompt Frame | **No frame** | Cleanest |
| Show current time | Either | Taste |
| Transient Prompt | **Yes** | Old commands shrink to a bare arrow, which keeps a long Claude session readable |
| Instant Prompt | **Verbose** | The recommended default |
| Apply changes | **Yes** | Otherwise nothing is saved |

**You should see** `New config: ~/.p10k.zsh` and your prompt redraw in the style you picked.

If the wizard never appears, run it yourself:

```bash
p10k configure
```

---

## Where your repos live

**Every Windows machine here uses the same path**, so nobody has to ask where somebody else keeps
their repos and every doc can print it literally:

```
~/arus/code/arus-innovation/arus-onboarding
~/arus/code/humini-ecosystem/<product>        once a product repo is granted to you
```

That is `~/arus/code`, one folder per org inside it. (`~` is shorthand for your home folder,
`/home/<your-username>`.) The installer defaults to it, your profile states it, and the `hum` and
`arus` aliases jump straight into the two org folders.

**Why it is not `C:\arus\code`.** That is the Windows filesystem, which WSL reaches as `/mnt/c`, and
working there costs you real things rather than theoretical ones: git and `npm install` are several
times slower because every file read crosses the filesystem boundary, and Linux file modes are not
preserved, so the executable bit on our own scripts is lost. The installer refuses a `/mnt` root
outright rather than letting someone discover this a week later.

**Windows still gets one fixed way in.** Two options, and the installer prints both with your real
paths filled in:

| | How | Needs |
|---|---|---|
| **Explorer path** | Paste `\\wsl.localhost\Ubuntu\home\<you>\arus\code` into the Explorer address bar, or run `explorer.exe .` from inside any repo | nothing |
| **A real `C:\arus\code`** | A symlink pointing at that same UNC path, so the folder exists on `C:` while the files stay on Linux | PowerShell as Administrator, once |

Read, browse and drag files through either. What you must not do is **copy** repos onto `C:` and work
on the copy: it diverges from the real one silently, which is a worse problem than a slow `git
status`.

Being straight about what has been tested: the WSL and Ubuntu side of this has been run end to end,
the Windows symlink has not been tried on one of our laptops yet. If it is refused on yours, pin the
UNC path to Quick Access instead, and say so, because it is one line in this doc either way.

---

## Part 3: linking your Claude seat

About 5 minutes.

```bash
claude
```

**First it asks which login method.** Pick **1, Claude account with subscription**. Not the Console
option, not the third-party one. Our seats are subscription seats.

**Then it opens a browser to sign in.** Use **your Arus email**, the same address the seat was
invited to. If no browser opens, it prints an address for you to open yourself, same as in 2.4.

**Then it asks whether you trust the files in this folder.** Yes, for our own repos. It asks once per
folder, not every time.

**You should see** a welcome panel. The line that matters is:

```
Sonnet 5 - Claude Team - Arus Innovation Pte Ltd
```

**That "Arus Innovation Pte Ltd" is your confirmation.** If it names a personal account instead,
type `/logout`, then `/login`, and pick your Arus address. Typing `/status` shows the same
information at any time.

The seat is yours. Do not share a login: seats are per person, and the whole point of one per person
is that your context, your history and your `CLAUDE.md` are yours.

Then talk to it normally. No special syntax, just sentences:

```
what is in this folder?
explain what this repo is for
what are the house rules I am expected to follow?
```

**Three things to know on day one:**

- **Esc** interrupts it mid-answer. Use it early and often.
- **`/exit`** quits back to your normal terminal. **`/help`** lists everything else.
- **The box at the bottom is a chat box, not a command line.** Typing `cd somewhere` there sends
  Claude a sentence, it does not move your terminal. To run commands yourself, `/exit` first. This
  one catches nearly everybody once.

New to the terminal generally? Read `docs/WELCOME-BEGINNER.md` in this repo next. It is written for a
Mac, so ignore the Ghostty and Homebrew lines, and everything else applies exactly.

---

## Part 4: the repo, and your first PR

About 10 minutes. This is the real finish line.

You have one repo, and it is this one. That is on purpose: it is the repo that just set up your
machine, it carries the rules you are about to be held to, and a mistake in it costs nothing.

Two lines, one at a time:

```bash
cd ~/arus/code/arus-innovation/arus-onboarding
```

```bash
claude
```

**Check the welcome panel names this folder**, `~/arus/code/arus-innovation/arus-onboarding`, and not
`/home/<you>`. If it names your home folder, you started Claude before moving into the repo. Type
`/exit` and run the two lines again, in order, pressing Enter after each.

Now give Claude a real first task, which you need anyway. Type this into Claude, replacing
`<myname>` with your own first name in lowercase:

```
copy profiles/windows-starter.env to profiles/<myname>.env, set my name and email,
and explain what each setting does. Then ship it.
```

**Two things to watch while it works, and they are the whole point of doing this task:**

**One: `GIT_NAME` is your display name, not your GitHub username.** It becomes the name on every
commit you ever make here. Other profiles read "Sundaran V" and "Vidhya Jayakumar", so yours should
read the same way. If Claude shows you a diff with your GitHub username sitting in `GIT_NAME`, tell
it plainly:

```
GIT_NAME should be my display name, not my GitHub username. Set it to "Your Name"
and show me the diff again.
```

**Two: read the diff before you approve it.** Claude shows you every file it is about to change and
every command it is about to run, and waits. That pause is deliberate. Approving without reading is
exactly the habit this first task exists to prevent. When you are happy, choose **Yes**.

**"ship it"** runs the `ship` skill, which does the git work for you: makes a branch, commits, pushes
the branch, opens a pull request, and merges it. You do not need to memorise git commands.

**You should see**, at the end, something like:

```
Shipped: PR #16 (https://github.com/arus-innovation/arus-onboarding/pull/16)
is merged and squashed into main, branch deleted.
```

That is your first merged PR at Arus. Product repos come after this: one line in your new profile
plus one grant from an owner, which is "Getting more access later" below.

---

## Part 5: the by-hand list

The installer printed these at the end. None of them are failures. About 5 minutes.

### 5.1 Install the huddle skill

Its own installer asks which agent to install for, which a script cannot answer. Run this in your
normal terminal, not inside Claude (`/exit` first if Claude is running):

```bash
npx skills add muthuishere-agent-skills/huddle
```

Pick **Claude Code** when it asks. If it reports the skill is already installed and identical, that
is a complete answer, move on.

### 5.2 Fill in your personal CLAUDE.md

`~/.claude/CLAUDE.md` is the file Claude reads about you at the start of every session, in every
repo. Easiest way is to ask Claude to do it. Start Claude and type:

```
open ~/.claude/CLAUDE.md and help me fill in the "Who I am" section
```

Tell it your role and what you are strong at. The more specific that file is, the less Claude
guesses. Grow it over time by saying "add that to my CLAUDE.md" whenever you find yourself explaining
the same thing twice.

### 5.3 Optional: a C:\arus\code shortcut

Only if you want your repos to appear under `C:` in Explorer. The installer printed this command with
your real path already filled in. It needs PowerShell as Administrator, once.

If Windows refuses it, pin the `\\wsl.localhost\Ubuntu\home\<you>\arus\code` path to Quick Access in
Explorer instead and carry on. Same access, no admin needed.

### 5.4 Optional: a one-word shortcut for starting work

Every morning, starting work is two lines: move into the repo, then start Claude. If you would rather
type one word, create a shortcut:

```bash
echo "alias arusgo='cd ~/arus/code/arus-innovation/arus-onboarding && claude'" >> ~/.zshrc
```

Close the terminal, open a new one, and from then on typing `arusgo` does both.

**Do not** put `cd` and `claude` directly into `~/.zshrc` instead. That variant launches Claude in
**every** terminal tab you ever open, including the ones where you only wanted to run a quick git
command, and you then have to quit Claude before you can do anything else.

### 5.5 You are done

You now have WSL working, Ubuntu set up, a Claude seat on the Arus organisation, one repo, and a
merged pull request with your name on it. Go back to the checklist at the top and tick off the last
box.

**Tomorrow morning, starting work is:** open Windows Terminal, then

```bash
cd ~/arus/code/arus-innovation/arus-onboarding
claude
```

You will not have to sign in again.

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
| `wsl --install` says a distribution already exists | Not an error. Ubuntu is registered already. Run `wsl --list --verbose` and use the table in 1.3 |
| You are in "Windows PowerShell (x86)" and unsure | It is the 32-bit build. Prefer the one without (x86), but the `wsl` commands here were tested from it and work. Not worth restarting over |
| PowerShell commands fail oddly, title bar does not say Administrator | You are not elevated. Close it, right-click the Start button, choose Terminal (Admin) |
| A new tab opens PowerShell instead of Ubuntu | The default profile is still PowerShell. Fix it in 1.6. Meanwhile, the dropdown arrow next to `+` opens Ubuntu |
| `command not found: claude` | Close the terminal, open a new one. New tools need a fresh shell. The single most common first-morning issue |
| A check in Part 2 failed: `which zsh` is blank | `sudo apt install -y zsh`, then also do the next row |
| A check in Part 2 failed: your `/etc/passwd` line does not end in `/usr/bin/zsh` | `sudo chsh -s "$(which zsh)" "$(whoami)"`, then close the terminal and open a new one |
| A check in Part 2 failed: `~/.oh-my-zsh/oh-my-zsh.sh` is missing | Restore the framework without touching your theme folder. Three lines: `git clone --depth=1 https://github.com/ohmyzsh/ohmyzsh.git /tmp/ohmyzsh` then `find /tmp/ohmyzsh -maxdepth 1 -mindepth 1 ! -name custom -exec cp -r {} ~/.oh-my-zsh/ \;` then `rm -rf /tmp/ohmyzsh` |
| Prompt is still a plain `$` after the installer and a new terminal | zsh is not your login shell. The two rows above, in order |
| `Unable to locate package wslu` | Not available on your Ubuntu release. Skip it. Sign-in web addresses get copied into your browser by hand instead, twice, ever |
| An `apt install` fails and installs none of the list | `apt` is all or nothing. Install the list again without the one package it could not find, then deal with that package separately |
| Prompt shows empty boxes or question marks instead of icons | The MesloLGS NF font is not installed on Windows, or not selected in Windows Terminal. Go back to 1.7 |
| You typed `cd ...` or `claude` into Claude's own input box | That box is a chat box, not a command line. `/exit` first, then run shell commands yourself |
| You pasted several commands at once and the output is jumbled | Windows Terminal ran them together. Run them one line at a time, pressing Enter after each |
| Nothing appears while typing a password | Normal on Linux. Type it and press Enter |
| Reaching your Linux files from Windows | Explorer address bar: `\\wsl.localhost\Ubuntu\home\<you>\arus\code` (older builds: `\\wsl$\Ubuntu\...`). Read and drag files freely. Just do not point a Windows Git or a Windows editor at a Linux repo |
| `sudo` asks for a password you do not know | It is the **Linux** password you created when Ubuntu first started, not your Windows login |
| Whole files show as changed in `git diff` | A Windows tool rewrote the line endings to CRLF. Edit Linux files with VS Code plus the WSL extension, or from inside Ubuntu, and this never happens |
| `npm install -g` fails with EACCES | The installer moves npm's global prefix to `~/.npm-global` so no global install needs sudo. If you hit this, you are in a shell that predates the change: open a new terminal. Do not fix it with `sudo npm` |
| Certificate or TLS errors after the laptop sleeps | The WSL clock drifted. `sudo hwclock -s` resets it |
| Ubuntu feels slow, or eats memory | WSL2 grows its memory as needed and does not always hand it back. Create `C:\Users\<you>\.wslconfig` with `[wsl2]` and `memory=8GB`, then run `wsl --shutdown` in PowerShell. Also exclude your WSL folders from third-party antivirus, which is a common cause of slow git |
| A repo will not clone, or you get a 404 | That is a grant that has not happened yet, not something you broke. Check your email for GitHub invites, accept them, then re-run with `--repos-only` |
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

This page was written from a real run, start to finish, on **25 August 2026**:

| | Version |
|---|---|
| Windows | Windows 11 |
| WSL | WSL2 |
| Ubuntu | 26.04 LTS "Resolute Raccoon" |
| Node | v22.23.2, installed by the installer |
| Claude Code | 2.1.241 |
| GitHub CLI | 2.46.0 |
| Result | merged PR on `main` |

Two things on this page have **not** been run on one of our laptops, and are marked optional where
they appear:

- The `C:\arus\code` symlink in 5.3, which has a stated fallback.
- A first install on Ubuntu 24.04 or older, including the older-Ubuntu `gh` workaround, which is
  carried over from the previous version of this document.

If you hit something this page does not cover, add the row. One line in the traps table is worth more
than the next person rediscovering it.
