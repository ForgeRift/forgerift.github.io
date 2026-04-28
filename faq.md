# ForgeRift Plugin FAQ

**For:** new users, support triage, and Anthropic marketplace reviewers  
**Products covered:** local-terminal-mcp · vps-control-mcp  
**Last updated:** April 2026

---

## General

**What is ForgeRift?**  
ForgeRift LLC builds audited Claude Desktop plugins that give Claude controlled, security-hardened access to real infrastructure — your local Windows machine (local-terminal-mcp) and your Linux VPS (vps-control-mcp). Every command passes through a three-tier security classifier before execution. Every call is audit-logged.

**Who is this for?**  
Developers, technical founders, and builders who are already using Claude daily and are tired of copying and pasting commands back and forth. local-terminal-mcp is aimed at Windows users with Claude Desktop; vps-control-mcp is for anyone running a Linux VPS they want Claude to help manage.

**Are these plugins open source?**  
Yes. Both plugins are released under the MIT License. Source is available at github.com/ForgeRift/local-terminal-mcp and github.com/ForgeRift/vps-control-mcp.

**Does ForgeRift work with Claude.ai (the web version)?**  
No. Both plugins require Claude Desktop (Windows). The .mcpb extension format is a Claude Desktop feature. The web interface does not support MCP plugins.

---

## Compatibility & Requirements

**What operating systems are supported?**  
local-terminal-mcp: Windows 10 and Windows 11 only. macOS and Linux support is planned but not yet available.  
vps-control-mcp: Runs on Ubuntu 20.04+ or Debian 11+ on your Linux VPS. You access it from Claude Desktop on Windows.

**Do I need a Claude Pro subscription?**  
You need Claude Desktop, which requires a Claude subscription. A Claude Pro plan is sufficient. Claude Team and Enterprise plans also work.

**Do I need an Anthropic API key?**  
An API key is optional but recommended. It enables Layer 3 AI-assisted review of AMBER-tier commands — a second pass where Claude itself evaluates borderline commands before they execute. Without it, the plugin still works fully, but AMBER commands skip the AI review layer. API keys are available at console.anthropic.com.

**Can I use this with Cowork mode?**  
Yes. Both plugins are compatible with Claude Cowork mode (the Claude desktop app).

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
Go to **Settings → Extensions** in Claude Desktop and remove the extension. No residual files or services are left behind.

---

## Security — What Claude Can and Cannot Do

**What can Claude do with local-terminal-mcp installed?**  
Claude can list directories, read files, search file contents, check git status, run npm commands (install, ci, list, run script), get system information, and run shell commands that pass the security classifier.

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

There are 140+ blocked patterns across 27 categories in local-terminal-mcp. The full list is documented in COMMANDS.md in the GitHub repository.

**What is the three-tier security model?**  
Every command Claude tries to run is classified before execution:

- **RED** — Hard-blocked. Permanently denied with no override, no flag, no escape. Returns a structured error explaining the category and reason.
- **AMBER** — Warning-required. The first call forces a dry run with a visible warning. A second explicit call is required to actually execute. Covers moderately risky but legitimate operations like bulk copies and wildcard renames.
- **GREEN** — Allowed and logged. Passes rate limiting, timeout cap, and audit logging. The default tier for all structured tools and safe shell commands.

**Can Claude access my passwords or secret files?**  
No. Sensitive file paths are blocked even from read-only tools. This includes .env files, SSH keys, .pem/.key/.pfx certificates, Windows credential stores, cloud credentials (.aws/, .gcloud/, .azure/), browser login data, kubeconfig, and more.

**Does the plugin send my data anywhere?**  
The plugin runs entirely on your machine and binds to localhost (127.0.0.1) — it is not reachable from the network. Audit logs are written to disk locally. The only external calls are to the Claude API (via Claude Desktop, to process your conversation) and to ForgeRift's license validation endpoint (to verify your subscription). No command output, file contents, or audit log data is transmitted to ForgeRift servers.

**Where are the audit logs stored?**  
In the `logs/audit.log` file within the plugin's install directory, managed by Claude Desktop. Log rotation caps at 10MB with one backup. Secret values (tokens, keys, passwords) are automatically redacted from logs before writing.

**Has this plugin been security-audited?**  
Yes. The plugin was tested against 80+ adversarial scenarios covering prompt injection, path traversal, Unicode homoglyphs, newline injection, chained commands, and category bypass attempts. The findings informed the current 140+ pattern blocklist. The full security model is documented in SECURITY.md.

---

## Pricing & Billing

**What does it cost?**  
- **Individual:** $14.99/month or $149/year (one plugin of your choice)
- **Bundle:** $19.99/month or $199/year (both local-terminal-mcp and vps-control-mcp)
- **Founder Cohort:** $9.99/month individual / $14.99/month bundle, locked for life — available to the first 100 subscribers or for 3 months post-launch, whichever comes first

**Is there a free trial?**  
Yes. All plans include a 14-day free trial. You will not be charged during the trial period.

**What payment methods are accepted?**  
Credit and debit cards via Stripe. Manage your subscription and billing at the link in your account confirmation email.

**Can I switch between monthly and annual billing?**  
Yes. You can switch at any time through the billing portal. Annual billing is not available on the Founder Cohort plan — monthly only.

**What happens if I cancel?**  
Cancellation takes effect at the end of your current billing period. You retain access through the period you've already paid for. No partial refunds are issued after the trial ends.

**What is the Founder Cohort rate lock?**  
Founder Cohort subscribers lock in their discounted monthly rate for the life of their subscription. The rate lock is tied to continuous active subscription — canceling permanently forfeits the Founder price. Reactivating after a lapse restores access at the then-current standard price, not the Founder rate. This cannot be reinstated.

**Are refunds available?**  
No refunds are issued after the trial period ends. ForgeRift may issue refunds at its discretion in cases of billing error or extraordinary circumstances. Contact support@forgerift.io with the subject "Refund Request."

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
This is expected in some situations — Claude's behavior is probabilistic and it occasionally defaults to explaining rather than acting. This is more common for commands it's uncertain about. To encourage direct execution, try: *"Use the run_command tool to run this directly — don't ask me to paste it."* We log and mitigate every instance of unnecessary passthrough as a defect.

**The plugin was working and stopped after a Claude Desktop update.**  
Claude Desktop updates can occasionally affect extension compatibility. Check for a plugin update first (you'll receive an email when one is available). If no update is available, email support@forgerift.io with your Claude Desktop version number and the error you're seeing.

**Where do I find error details if something goes wrong?**  
The audit log at `logs/audit.log` inside the plugin's install directory records every tool call and its result, including error details. Claude can read this file directly — ask: *"Check the audit log and tell me what's failing."*

---

## Support

**How do I get help?**  
Email support@forgerift.io. Response is best-effort — we're a small team and do not commit to a specific turnaround time.

**Can I report a bug or request a feature?**  
Yes — GitHub Issues at github.com/ForgeRift/local-terminal-mcp is the best place for bugs and feature requests. This keeps a public record and lets other users follow along.

**How do I report a security vulnerability?**  
Email security@forgerift.io. Do not open a public GitHub Issue for security vulnerabilities.

**Is there documentation beyond this FAQ?**  
Yes. The GitHub repository includes:
- **GETTING_STARTED.md** — step-by-step setup for new users
- **CLAUDE_CONTEXT.md** — load into a Claude Project so Claude always knows how the plugin works
- **COMMANDS.md** — full breakdown of GREEN/AMBER/RED commands by category
- **TROUBLESHOOTING.md** — common issues and fixes
- **SECURITY.md** — full security model and configuration reference

---

*ForgeRift LLC · forgerift.io · support@forgerift.io*
