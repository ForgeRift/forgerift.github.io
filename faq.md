# ForgeRift Plugin FAQ

**For:** new users, support triage, and Anthropic marketplace reviewers  
**Products covered:** local-terminal-mcp · vps-control-mcp  
**Last updated:** 2026-04-29

---

## General

**What is ForgeRift?**  
ForgeRift LLC builds security-hardened Claude Desktop plugins that give Claude controlled access to real infrastructure — your local Windows machine (local-terminal-mcp) and your Linux VPS (vps-control-mcp). Arbitrary shell commands pass through a three-tier security classifier (RED/AMBER/GREEN); structured tools enforce sensitive-file guards. Every call is audit-logged.

**Who is this for?**  
Developers, technical founders, and builders who are already using Claude daily and are tired of copying and pasting commands back and forth. local-terminal-mcp is aimed at Windows users with Claude Desktop; vps-control-mcp is for anyone running a Linux VPS they want Claude to help manage.

**Are these plugins open source?**  
Yes. Both plugins are released under the MIT License. Source is available at github.com/ForgeRift/local-terminal-mcp and github.com/ForgeRift/vps-control-mcp.

**Is ForgeRift affiliated with Anthropic?**  
No. ForgeRift LLC is an independent third-party developer. We are not affiliated with, endorsed by, or sponsored by Anthropic PBC. The plugins extend Claude Desktop, which is an Anthropic product, but ForgeRift has no relationship with Anthropic beyond using their published platform.

**Does ForgeRift work with Claude.ai (the web version)?**  
No. Both plugins require Claude Desktop. The .mcpb extension format is a Claude Desktop feature; the web interface does not support MCP plugins. (local-terminal-mcp is Windows-only; vps-control-mcp can be used from Claude Desktop on Windows or macOS.)

---

## Compatibility & Requirements

**What operating systems are supported?**  
local-terminal-mcp: Windows 10 and Windows 11 only. macOS and Linux support is planned but not yet available.  
vps-control-mcp: The server component runs on Ubuntu 20.04+ or Debian 11+ on your Linux VPS. You access it from Claude Desktop on Windows or macOS — any OS that runs Claude Desktop can connect to the remote VPS.

**Do I need a Claude Pro subscription?**  
You need Claude Desktop, which is a free download from claude.ai/download. A Claude subscription (Pro, Team, or Enterprise) is required for full usage — the free tier has limited message capacity.

**Do I need an Anthropic API key?**  
No — it is optional. If provided, it enables AI-assisted safety classification for **every** shell command submitted via `run_command` — not only AMBER-tier commands. The command text and user-provided justification are sent to Anthropic's API for classification before execution; no environment variables, working directory, or other system context is included. A high-risk classification may independently block execution — the AI review is not purely advisory. Each API call consumes tokens billed to your Anthropic account at Anthropic's rates — ForgeRift does not control or receive these charges. This uses your API account and is subject to Anthropic's data handling. Without it, the AI classification layers are skipped entirely and AMBER commands require a manual dry-run-and-confirm instead. If the Anthropic API call fails (network error, rate limit, or invalid key), the AI layers fall back automatically to the manual dry-run-and-confirm flow; the failure reason is shown in the dry-run output. API keys are available at https://console.anthropic.com.

**Why does Claude ask me to describe the task every time I run a shell command?**  
The `run_command` tool requires a `justification` string — a plain-English description of what you're trying to accomplish and why the structured tools (like `run_git_command` or `run_npm_command`) can't cover it. This is a required parameter, not optional. It serves two purposes: it feeds the AI-assisted safety classification on every `run_command` invocation (if you have an Anthropic API key configured) and it creates a human-readable audit trail. If you have an API key configured, your justification text is sent to Anthropic's API alongside the command text — avoid including secrets or sensitive data in your justification.

For AMBER-tier commands specifically, the first call always runs as a dry-run preview (`dry_run=true` is the default). After reviewing the dry-run output and any AI evaluation result presented alongside it, Claude will re-invoke the tool with `dry_run=false` to actually execute — but only after you confirm in chat. You are always in control of whether execution proceeds.

**Can I use this with Cowork mode?**  
Yes. Both plugins are compatible with Claude Cowork mode *(Cowork mode is an optional Anthropic feature for autonomous multi-step tasks — the plugin works in both standard Claude Desktop and Cowork mode.)*. AMBER-tier commands still require explicit user confirmation in Cowork mode — Claude pauses and presents the dry-run description before executing.

---

## Installation

**How do I install local-terminal-mcp?**  
After subscribing at forgerift.io, you'll receive a `local-terminal.mcpb` file and a license key by email. In Claude Desktop, go to **Settings → Extensions → Install Extension**, select the .mcpb file, and enter your license key when prompted. No terminal, no config files, no Windows Service required.

**How long does installation take?**  
Usually under two minutes from .mcpb file to Claude responding to your first command.

**How do I verify it's working after install?**  
Open a new Claude Desktop conversation and ask: *"Can you list the files in my Documents folder?"* If Claude responds with an actual file listing, you're connected. If it says it can't access your files, see the Troubleshooting section below.

**How do I update the plugin?**  
Updates are managed through Claude Desktop's Extensions settings — no terminal commands needed. When a new version is released, you'll receive an email with the updated .mcpb file.

**How do I uninstall?**  
Go to **Settings → Extensions** in Claude Desktop and remove the extension. The plugin runs no Windows Service and installs no files outside the extension directory. The audit log (`logs/audit.log` and `logs/audit.log.old`) lives inside the extension's install directory. Claude Desktop may not delete the extension's install directory automatically on uninstall — to remove all traces, delete that directory manually after removing the extension. If you copied the audit log elsewhere for your own records, delete that copy yourself.

---

## Security — What Claude Can and Cannot Do

**What can Claude do with local-terminal-mcp installed?**  
Claude can list directories, read files, search file contents, run read-only git commands (status, log, diff, branch, show, stash list, tag, rev-parse, ls-files), run read-only npm commands (list, ls, outdated, audit, view, why, explain), get system information, and run shell commands that pass the security classifier. Note: git fetch, npm install, npm ci, and npm run are not available through the plugin — run them in your own terminal.

**What is Claude permanently blocked from doing?**  
The following are hard-blocked and cannot be overridden by any instruction:
- Deleting files or directories
- Shutting down, rebooting, or suspending the system
- Installing or removing software packages
- Making outbound network calls (curl, wget, ssh, scp, Invoke-WebRequest)
- Accessing passwords, SSH keys, .env files, cloud credentials, or browser login data
- Running arbitrary PowerShell or bash via code execution (Invoke-Expression, -EncodedCommand, bash -c, etc.)
- Privilege escalation (runas, sudo)
- Creating scheduled tasks or services
- Modifying the registry in destructive ways

There are 140+ blocked patterns across 27 categories in local-terminal-mcp. The full categorized list is published in COMMANDS.md in the public GitHub repository — exact counts may vary slightly between releases as patterns are added or refined; the CHANGELOG records changes.

**What is the three-tier security model?**  
Every command Claude tries to run is classified before execution:

- **RED** — Hard-blocked. Permanently denied with no override, no flag, no escape. Returns a structured error explaining the category and reason.
- **AMBER** — Warning-required. The first call forces a dry run with a visible warning. A second explicit call is required to actually execute. Covers moderately risky but legitimate operations like bulk copies and wildcard renames.
- **GREEN** — Allowed and logged. Subject to a per-tool wall-clock timeout (30 seconds for `run_command` and `run_git_command`, 60 seconds for `run_npm_command`) and full audit logging. The default tier for all structured tools and safe shell commands.

**Can Claude access my passwords or secret files?**  
No. Sensitive file paths are blocked even from read-only tools. This includes .env files, SSH keys, .pem/.key/.pfx/.ppk certificates, Windows credential stores, cloud credentials (.aws/, .gcloud/, .azure/), browser login data, kubeconfig, and more.

**Does the plugin send my data anywhere?**  
The plugin runs entirely on your machine as a sandboxed child process spawned by Claude Desktop over stdio — it opens no inbound network port and is not reachable from the network. Audit logs are written to disk locally. The only external calls are:

1. **To the Claude API** (via Claude Desktop): your conversation — including any command output that Claude reads as context — is processed by Anthropic per their privacy policy. ForgeRift does not receive this data.
2. **To ForgeRift's license validation endpoint** (hosted on Supabase): your license key is sent at startup to verify your subscription. The key is transmitted as a URL query parameter over HTTPS, which means it may appear in server-side access logs on ForgeRift's infrastructure. See the [Privacy Policy](https://forgerift.io/privacy.html) §2.1 for retention details.
3. **To Anthropic's API** (optional, only if you supply an Anthropic API key): the command text and user-provided justification for **every** `run_command` invocation — not just AMBER-tier commands — are sent to Anthropic's API for AI-assisted safety classification before execution. A high-risk result may independently block execution. This is opt-in — without an API key the AI classification layers are skipped entirely.

No command output, file contents, or audit log data is transmitted to ForgeRift servers.

**Where are the audit logs stored?**  
In `logs/audit.log` within the extension's install directory (managed by Claude Desktop). When the file reaches 10 MB it is renamed to `audit.log.old`, overwriting any prior backup; maximum on-disk storage is approximately 20 MB at any time. Secret values (tokens, keys, passwords) are automatically redacted from logs before writing.

**What security testing has been done on this plugin?**  
The plugin has been internally reviewed across multiple rounds of adversarial testing covering prompt injection, path traversal, Unicode homoglyphs, newline injection, chained commands, and category bypass attempts. Findings are tracked in ADVERSARIAL_REVIEW.md; each finding maps to a regression test added to the suite. These scenarios drove the core blocklist; additional patterns were derived from public threat intelligence, bringing the total to 140+ blocked patterns across 27 categories. No independent third-party audit has been conducted. The full security model is documented in SECURITY.md.

---

## Pricing & Billing

**What does it cost?**  
- **Individual:** $14.99/month or $149/year (one plugin of your choice)
- **Bundle:** $19.99/month or $199/year (both local-terminal-mcp and vps-control-mcp; each installs separately as its own .mcpb extension — note that local-terminal-mcp is Windows-only, so macOS/Linux users on the Bundle plan can only use the vps-control-mcp half)
- **Founder Cohort (limited):** $9.99/month individual / $14.99/month bundle *(Bundle Founder pricing equals the regular Individual plan rate)* — rate-locked as long as your subscription stays active, monthly billing only — eligibility closes at the earlier of (a) the 100th paid subscriber or (b) 3 months after the marketplace listing date

**Is there a free trial?**  
Yes. All plans include a 14-day free trial. A valid payment method is required at sign-up but will not be charged until the trial ends. Your card issuer may show a temporary $0 or $1 authorization hold at sign-up, which is automatically released and does not constitute a charge. You can cancel at any time during the trial to avoid any charge.

**What payment methods are accepted?**  
Credit and debit cards via Stripe. Manage your subscription and billing at the link in your account confirmation email.

**Can I switch between monthly and annual billing?**  
Yes. You can switch at any time through the Stripe Customer Portal. Annual billing is not available on the Founder Cohort plan — monthly only. If you are a Founder Cohort member and accidentally select annual billing, contact support@forgerift.io within 7 calendar days to revert to the monthly Founder rate; ForgeRift will issue a prorated refund and restore your rate lock — see Terms Section 6.8 for details.

**What happens if I cancel?**  
Cancellation takes effect at the end of your current billing period. You retain access through the period you've already paid for. No partial refunds are issued after the trial ends.

**What is the Founder Cohort rate lock?**  
Founder Cohort subscribers lock in their discounted monthly rate for as long as the subscription remains continuously active. The rate lock is tied to continuous active subscription — voluntarily canceling permanently forfeits the Founder price, and reactivating after such a lapse restores access at the then-current standard price. This cannot be reinstated. Brief involuntary suspensions caused by payment-method failure (such as an expired card) do not count as a lapse if fully cured within 30 days of the initial suspension event — see Terms Section 6.8 for details.

**Are refunds available?**  
For confirmed ForgeRift billing errors (duplicate charges, charges after timely cancellation, charges at an incorrect rate), ForgeRift will refund the incorrect amount within 10 business days — see Terms Section 6.5. For extraordinary circumstances outside billing errors, ForgeRift may issue refunds at its discretion. No other refunds are issued after the trial period ends. Contact support@forgerift.io with the subject "Refund Request."

**What happens to my data when I cancel?**  
Short answer: most data is gone within 90 days; billing records are kept 7 years for tax compliance, which is standard practice.

Your license key is deactivated within 24 hours of cancellation. Audit logs are stored locally on your machine — they are not transmitted to ForgeRift, so you retain control of them and can delete them yourself. Your account and billing records (email address, Stripe billing history) are retained for 7 years for accounting compliance, then deleted. Email records held by Resend are retained for approximately 90 days. License validation logs (which contain only your license key and a timestamp) are deleted 90 days after each record is created. Because the plugin validates on each startup, an active subscriber has a rolling 90-day window of records on file; after cancellation and license deactivation, all remaining records are deleted within 90 days. You can request deletion of your personal data by emailing support@forgerift.io — see the [Privacy Policy](https://forgerift.io/privacy.html) for the full retention schedule.

---

## Troubleshooting

**Claude says it can't access my files after installation. What's wrong?**  
The most common causes:
1. The extension did not finish installing — go to Settings → Extensions and confirm it shows as active
2. The license key was entered incorrectly — reinstall the extension and re-enter the key carefully
3. You're in a Claude.ai browser session, not Claude Desktop — the plugin only works in the desktop app
4. Claude Desktop needs a restart after a fresh install — close and reopen it

**I'm getting a RED block on a command I think should be allowed. What do I do?**  
RED blocks are permanent and cannot be overridden by Claude or by you. If the command is legitimately blocked (it's in one of the 27 blocked categories), you'll need to run it yourself in your own terminal. If you believe the block is a false positive — a safe command being caught by an overly broad pattern — open a GitHub Issue at github.com/ForgeRift/local-terminal-mcp/issues with the command and context. We treat false positives as defects.

**Claude keeps suggesting I paste commands into my terminal instead of running them directly.**  
This is expected in some situations — Claude's behavior is probabilistic and it occasionally defaults to explaining rather than acting. This is more common for commands it's uncertain about. To push past it, say: *"Use the run_command tool directly"* — Claude will comply. We're actively working to reduce this through prompt improvements shipped with each release. We log and mitigate every instance of unnecessary passthrough as a defect.

**The plugin was working and stopped after a Claude Desktop update.**  
Claude Desktop updates can occasionally affect extension compatibility. Check for a plugin update first (you'll receive an email when one is available). If no update is available, email support@forgerift.io with your Claude Desktop version number and the error you're seeing.

**Claude says "Subscription check timed out" or "Network error during subscription check."**  
The plugin validates your license key every time it starts. If the ForgeRift license server is unreachable (network outage, server maintenance, or VPN/firewall blocking outbound HTTPS), the plugin **fails closed** — it exits immediately and Claude loses access to all plugin tools. Fail-closed was chosen deliberately: a tool with shell access to your machine should never silently fall back to an unverified state. ForgeRift operates the validation endpoint on dedicated infrastructure and treats its uptime as a product commitment. There is no offline grace period. Wait a few minutes and restart Claude Desktop to retry. If the error persists, check your internet connection and verify forgerift.io is reachable. Email support@forgerift.io if the problem continues.

**Where do I find error details if something goes wrong?**  
The audit log at `logs/audit.log` inside the extension's install directory records every tool call and its result, including error details. Claude can read this file directly — ask: *"Check the audit log and tell me what's failing."*

---

## Support

**How do I get help?**  
Email support@forgerift.io. Response is best-effort — we're a small team and do not commit to a specific turnaround time.

**Can I report a bug or request a feature?**  
Yes — GitHub Issues at github.com/ForgeRift/local-terminal-mcp is the best place for bugs and feature requests. This keeps a public record and lets other users follow along.

**How do I report a security vulnerability?**  
Email security@forgerift.io. Do not open a public GitHub Issue for security vulnerabilities.

**Is there documentation beyond this FAQ?**

Yes — full documentation lives in the GitHub repository at [github.com/ForgeRift/local-terminal-mcp](https://github.com/ForgeRift/local-terminal-mcp):

- **README.md** — overview, quick-start, and configuration reference
- **GETTING_STARTED.md** — step-by-step install guide for non-developers  
- **COMMANDS.md** — complete RED/AMBER/GREEN pattern reference
- **SECURITY.md** — threat model, safety guarantees, and operator controls
- **TROUBLESHOOTING.md** — common issues and fixes
- **CHANGELOG.md** — full release history
- **CREDITS.md** — open-source dependencies and attributions

---
*© 2026 ForgeRift LLC · [forgerift.io](https://forgerift.io) · [support@forgerift.io](mailto:support@forgerift.io)*
