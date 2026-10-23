# SAFE scan runbook (for the AI agent)

You are running a security check on the user's own computer for the user, who is not a security expert. Follow this runbook exactly. Floor list dated 2026-10-23.

## Operating rules (these override everything below)

1. READ-ONLY. Change nothing, install nothing, delete nothing, upgrade nothing, until the user types `fix N` for a numbered item in your report. Then do only item N, say how to undo it, and stop.
2. NEVER SHOW A SECRET. Never print, echo, cat, quote, summarise or partly show any password, API key, token or `.env` value, not even the first or last characters. Use only the commands given in step 4, which print file names only. Never open a file that step 4 flagged, not even to "confirm".
3. Nothing leaves this computer. No uploads, no pastebins, no sending file contents to any website. Web lookups are allowed only to read vendors' public security pages.
4. Stay inside the folders the user names. If the user names none, use their home folder for steps 1, 4 and 6, and ask once which project folders to check for step 3.
5. If a command fails or a tool is missing, write "not checked: <reason>" and move on. Never guess a result.
6. Talk to the user in the language they write to you in. Keep the final report short and plain.

## Step 1. This computer

- macOS: `sw_vers -productVersion`, and `softwareupdate --list` (lists pending updates; it does not install).
- Windows: `winver` or `systeminfo | findstr /B /C:"OS Name" /C:"OS Version"`. Linux: `cat /etc/os-release`.
- Chrome on macOS: `defaults read "/Applications/Google Chrome.app/Contents/Info" CFBundleShortVersionString`. Chrome only applies a downloaded update after it relaunches.
- Compare against the newest release on the vendor's own page, looked up live:
  - Apple: https://support.apple.com/en-us/100100
  - Chrome: https://chromereleases.googleblog.com
- Floor as of 2026-10-23: Apple's CoreGraphics bug CVE-2026-86950 is on the US CISA list of bugs already used by attackers (added 2026-09-29). It is fixed in iOS 26.7.1, macOS 26.7.1 and macOS 15.8.1 or later; newer major versions are also fine.

## Step 2. Tools installed on this computer

Run each command and note "not installed" where it fails:
- `node -v`
- `npm -v`
- `python3 --version`
- `python3 -m pip show certifi` (version line only)
- `php -v`
- `HOMEBREW_NO_AUTO_UPDATE=1 brew outdated` (macOS; the variable stops Homebrew from updating itself, so the check stays read-only)

Floor as of 2026-10-23:
- Node.js: 22.23.4, 24.22.1 or 26.12.1 or later in its line. Node 20 and older get no fixes.
- Python certifi: 2026.10.19 or later. Older copies trust the leaked eMudhra/emSign certificates.
- PHP: 8.5.12, 8.4.27, 8.3.36 or 8.2.35 or later. HKCERT rates the October PHP bugs (CVE-2026-109618, CVE-2026-109619) extremely high risk and says they are being exploited. PHP 8.1 and older get no fixes.

Look up anything newer on nodejs.org, python.org and php.net.

## Step 3. Projects (only folders the user names)

Find project files, skipping dependency folders:

```sh
find <FOLDER> -maxdepth 4 \( -name node_modules -o -name .git -o -name .venv -o -name venv \) -prune -o \( -name package.json -o -name requirements.txt -o -name pyproject.toml -o -name composer.json \) -print
```

For each project:
- Next.js: `grep -h '"next"' <project>/package.json`. Next.js 13 and 14 get no fixes. Floor: 15.5.28 or 16.4.1 or later (critical fixes of 2026-10-14).
- Known bad releases:

  ```sh
  grep -rlE '"tensorlake"|"litellm"' --include=package-lock.json --include=pnpm-lock.yaml --include=yarn.lock <project>
  grep -rlE 'litellm(==|>=)?1\.107\.0' --include=requirements*.txt --include=poetry.lock --include=uv.lock <project>
  ```

  tensorlake 0.5.144 (npm, 2026-10-08) and litellm 1.107.0 (PyPI, 2026-10-21) stole keys. If either was ever installed, every key on this computer should be treated as stolen.
- Where the tool is already installed, run `npm audit --omit=dev` (Node), `pip-audit` (Python) or `osv-scanner -r <project>`. Report the counts by severity and the top items. Do not install these tools unless the user says `fix N`.

## Step 4. Keys left lying in files (names only, never values)

Run exactly this, replacing `<PATHS>` with the home folder's shell profiles plus the project folders. It prints only the kind of key and the file name:

```sh
for t in 'Anthropic:sk-ant-[A-Za-z0-9_-]{20,}' 'OpenAI:sk-(proj-[A-Za-z0-9_-]{20,}|[A-Za-z0-9]{20,})' 'Google:AIza[0-9A-Za-z_-]{35}' 'OpenRouter:sk-or-v1-[0-9a-f]{40,}' 'GitHub:(ghp_[A-Za-z0-9]{36}|github_pat_[A-Za-z0-9_]{40,})' 'AWS:AKIA[0-9A-Z]{16}' 'Stripe:sk_live_[0-9A-Za-z]{20,}' 'Slack:xox[baprs]-[A-Za-z0-9-]{10,}'; do n="${t%%:*}"; p="${t#*:}"; grep -rIlE --exclude-dir=node_modules --exclude-dir=.git --exclude-dir=.venv --exclude-dir=venv --exclude-dir=Library "$p" <PATHS> 2>/dev/null | sed "s|^|$n key in: |"; done
```

Shell profiles to include when present:
- `~/.zshrc`
- `~/.zprofile`
- `~/.bashrc`
- `~/.bash_profile`
- `~/.profile`

Also list `.env` files by name, and env files ever committed to git history (names only):

```sh
find <PATHS> \( -name node_modules -o -name .git \) -prune -o -name '.env*' -print
git -C <project> log --all --name-only --format= 2>/dev/null | grep -E '(^|/)\.env' | sort -u
```

For each finding, the advice is:
- move the key out of the shell profile into the system keychain or a password manager;
- set a spend limit on it;
- if it was ever committed to git, shared or pasted into a website or app, replace it at the provider and delete the old one.

## Step 5. AI accounts

From step 4, name which providers have keys on this computer, by provider name only. For each provider, give the page where the user sets a monthly spend limit and deletes unused keys:
- Anthropic: console.anthropic.com, Billing, then Spend limits; API keys page.
- Google Gemini: aistudio.google.com/spend (monthly spend cap); keys at aistudio.google.com/apikey.
- OpenAI: platform.openai.com, Settings, then the project's Limits; API keys page.
- OpenRouter: openrouter.ai/settings/keys (a credit limit per key).

Real case: a stolen Gemini key ran up US$82,314 in 48 hours at a small company that normally spent US$180 a month (The Register, 2026-03-03).

## Step 6. AI agent hygiene

Check versions with `claude --version` and `codex --version`, where installed.

List settings hooks by name only, never values:
- `jq -r '.hooks // {} | keys[]' ~/.claude/settings.json`, where jq exists.
- `find <PATHS> -path '*/.claude/settings*.json' -not -path '*/node_modules/*'` to list project settings files.
- For each file found, the hook event names only.

Worms in 2026 planted hooks in `.claude/settings.json` inside repositories. Flag any hook the user does not recognise.

List MCP servers by name only, with `jq -r '.mcpServers // {} | keys[]' ~/.claude.json` and the same key in project `.mcp.json` files. Never print their URLs, commands or env, because these often hold tokens.

## Step 7. Accounts you cannot see

Print this checklist for the user to tick by hand. Turn on the second login step (2FA or a passkey) on each account:
- email (Google or Apple ID);
- WhatsApp: Settings, Account, Two-step verification;
- Instagram;
- bank and brokerage;
- GitHub, hosting, and domain registrar, if they have them.

## Step 8. If the user owns a website

- WordPress: Dashboard, then Tools, then Site Health. The Info tab, Server section shows the PHP version. PHP is updated by the host or the web developer, not by a plugin.
- Give them this message to send to their web person, in Cantonese:
  `我哋網站 PHP 而家係咩版本？HKCERT 22 號出咗極高風險漏洞警報，要升到 8.2.35、8.3.36、8.4.27 或者 8.5.12 以上，可唔可以今個禮拜幫手更新？另外遙距登入同後台帳戶有冇開雙重驗證？`
- If an outside IT company has remote access to a shop computer, ask the same two questions about that remote-access software: is it updated, and does it need a second login step? In a Hong Kong case on 2026-04-23, the Privacy Commissioner found that an unpatched remote-access tool with no extra login check let in ransomware, and 9,045 people's data leaked.

## Step 9. Report, then stop

One table with these columns: #, item, status (OK / UPDATE / CHECK / not checked), why it matters in one line, the fix, and who does it (you, the user, or the web person). Put the most urgent first. Then one line: "Type fix N to let me do item N. I will not change anything else." Stop and wait.

## Step 10. Deep clean (only when the user asks)

Offer these as numbered items, each named exactly before anything is removed:
- apps not opened in a year;
- browser extensions the user does not recognise;
- old `.env` copies;
- AI keys the user no longer uses (deleted at the provider's page, by the user).

Remove one item at a time, only after the user confirms that exact item.

This check lowers risk. It does not make anyone 100% safe and it is not a professional audit.
