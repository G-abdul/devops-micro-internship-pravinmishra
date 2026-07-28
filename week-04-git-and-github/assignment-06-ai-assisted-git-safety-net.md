# Assignment 6 — Building an AI-Assisted Git Safety Net (PR Ready Check)

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In Week 2 you built Claude Code hooks that block a dangerous action *before* it happens (`PreToolUse`), and a restricted skill that could look but not touch (`allowed-tools` without `Write`). In this assignment you will discover that Git has the exact same idea, decades older: a **pre-commit hook** that blocks a commit before it's created.

You will build both halves of a real "PR Ready" workflow:

1. A **Git hook that follows fixed rules** — scans staged changes for hardcoded secrets and oversized files and refuses the commit. No AI involved, no guessing, just a rule that gives the same answer every time.
2. A **restricted Claude Code skill** (`/pr-ready`) that reads your staged diff and drafts a Pull Request title, description, and a short list of things worth a second look — the kind of judgment a fixed rule can't make (mixed changes, missing context, unclear intent). The skill never commits, pushes, or opens the PR. You do that yourself, using its draft as a starting point.

This mirrors the Agentic Loop from Week 3's Linux triage assignment: **Gather → Analyze → Human Act → Verify**. The hook and the skill both gather and analyze; only you act.

---

# Task 0 — Confirm Your Fork and Create a Feature Branch

## Goal

Confirm you are working in your own fork, then create a dedicated branch for this assignment.

### Evidence

#### Screenshot 1 — Output of git remote -v and git branch showing the new branch

![Screenshot-1](./screenshots/Screenshot-1-git-remote-v-new-branch.png)

---

### Notes

**1. Why create a dedicated branch instead of doing this work on main?**

A dedicated branch keeps my changes isolated from the main codebase while I work. It allows me to test, review, and make corrections without affecting the stable version of the repository. Using a separate branch also makes collaboration easier because changes can be reviewed through a Pull Request before being merged.
---

# Task 1 — Stage a Change With Realistic Risk

## Goal

On your own fork of this repository (the one you've been submitting your DMI work in since onboarding), create a new branch and stage a change that a real reviewer should catch: a hardcoded-looking secret and a leftover debug statement.

### Evidence

#### Screenshot 1 — Output of  `git status` showing the staged file on feature/ai-pr-ready

![Screenshot-1](./screenshots/git-status-scripts-staged.png)

---

### Notes

**1. Why does this assignment use an obviously fake key instead of a real one?**

The assignment uses a fake AWS key to safely simulate a real security risk without exposing actual credentials. Using a real key could create a serious security issue if it were accidentally committed to GitHub or shared publicly. The fake key allows us to test detection mechanisms while following security best practices.

---

# Task 2 — Write a Real Git Pre-Commit Hook

## Goal

Create a tracked, shareable pre-commit hook that blocks a commit containing secret-like patterns or files over 1MB.

### Evidence

#### Screenshot 2 — `hooks/pre-commit` open in VS Code showing the full script

![Screenshot-2](./screenshots/hooks-pre-commit-VS-Code.png)


---

#### Screenshot 3 — Output of `git config core.hooksPath` confirming it points to `hooks`

![Screenshot-3](./screenshots/git-config-core-hookspath.png)


---

### Notes

**1. Why is `hooks/pre-commit` tracked in the repo instead of living only in `.git/hooks/`?**

hooks/pre-commit is tracked in the repository so it can be shared with every team member and version-controlled alongside the project. If it only existed in .git/hooks/, it would remain local to one machine and would not be included when others clone the repository. Tracking it in the repo ensures everyone uses the same safety checks and follows the same standards before committing code.

---

**2. Compare this to `PreToolUse` from Week 2 Assignment 6. What does each one intercept, and what do they have in common?**

The pre-commit hook intercepts Git commits before they are created and checks staged files for issues such as secrets or oversized files. PreToolUse intercepts AI tool requests before a tool is executed and can allow, warn, or block the action.

---

# Task 3 — Prove the Hook Blocks the Risky Commit

## Goal

Attempt to commit the staged file from Task 1 and show the hook rejecting it.

### Evidence

#### Screenshot 4 — Terminal showing `git commit` rejected with the hook's "BLOCKED" message naming the exact file

![Screenshot-4](./screenshots/git-commit-BLOCKED-message.png)


---

### Notes

**1. Which line in `hooks/pre-commit` matched your fake key, and why did it match?**

The line that matched my fake key was:

if git diff --cached -- "$file" | grep -qE 'AKIA[0-9A-Z]{16}|-----BEGIN (RSA|OPENSSH|PRIVATE) KEY-----'; then

Specifically, the regular expression:

AKIA[0-9A-Z]{16}

matched the fake AWS access key in scripts/notify.sh:

AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP

It matched because the key starts with AKIA and is followed by 16 uppercase letters, which fits the pattern the hook was designed to detect. The hook scans the staged changes using git diff --cached and blocks the commit when it finds a string that resembles an AWS access key, even though the credential used in this assignment is intentionally fake.

---

**2. Could this hook have caught a poorly-named variable that stores a secret without the `AKIA` prefix? What does that tell you about the limits of a fixed rule like this?**

No, this hook would likely not catch a poorly named variable storing a secret if it did not match one of the specific patterns defined in the regular expression. For example, a variable such as:

DB_PASSWORD=mysecretpassword123

or

TOKEN=abc123xyz789

would probably pass through the hook because neither value matches the AKIA[0-9A-Z]{16} pattern or the private key patterns being searched for.

This demonstrates an important limitation of fixed-rule checks: they can only detect what they are explicitly programmed to look for. They are fast, consistent, and reliable for known patterns, but they cannot understand context or recognize every possible secret format. That's why fixed-rule hooks are best used alongside AI-assisted reviews and human judgment, which can identify risks that don't match predefined rules.

---

# Task 4 — Build the `/pr-ready` Skill

## Goal

Create a manually invoked Claude Code skill that reads your staged changes and produces a PR-readiness report and a draft PR description — without writing, committing, or pushing anything itself.

### Evidence

#### Screenshot 5 — `SKILL.md` frontmatter showing `allowed-tools: Bash, Read, Grep` (no `Write`) and `disable-model-invocation: true`

![Screenshot-5](./screenshots/SKILL.md-frontmatter-showing.png)


---

#### Screenshot 6 — `/pr-ready` output while the risky file is still staged, showing it flagged the secret and/or debug statement

![Screenshot-6](./screenshots/Screenshot-6-pr-ready-.png)


---

### Notes

**1. Why does `/pr-ready` have `Bash` and `Read` but not `Write`?**

1. Why does /pr-ready have Bash and Read but not Write?

/pr-ready has Bash and Read permissions because it needs to inspect the repository and analyze the staged changes without modifying anything. Bash allows the skill to run commands such as git diff --cached and git status, while Read allows it to view file contents and understand the changes being reviewed.

It does not have Write permission because its role is only to provide recommendations, not to make changes. This prevents the AI from editing files, creating commits, pushing code, or opening Pull Requests on its own. Keeping Write disabled ensures that all repository changes remain under human control, following the principle of "AI advises, humans decide and act." This reduces the risk of unintended modifications and keeps the engineer responsible for every Git operation.

---

**2. The pre-commit hook and `/pr-ready` both looked at the same staged diff. Did they flag the same things? What did one catch that the other didn't?**

They flagged some of the same issues, but they did so in different ways.

The pre-commit hook detected the credential-shaped string because it matched the hardcoded pattern AKIA[0-9A-Z]{16} and immediately blocked the commit. Its job was to enforce a strict rule and prevent risky code from being committed. However, it did not identify the debug statement because no rule was written to check for debug messages.

The /pr-ready skill also flagged the AWS access key, but it went further by explaining why it was risky and suggesting possible fixes. In addition, it identified the leftover debug statement:

echo "DEBUG: token is $AWS_ACCESS_KEY_ID"

which the pre-commit hook did not detect. It also reviewed the overall change and provided recommendations for cleaning up the code before opening a Pull Request.

---

# Task 5 — Fix the Issues and Re-Verify

## Goal

Remove the secret and debug statement, then prove both gates now pass clean.

### Evidence

#### Screenshot 7 — `git commit` succeeding after the fix (no BLOCKED message)

![Screenshot-7](./screenshots/git-commit-succeeding-no-BLOCKED-message.png)


---

#### Screenshot 8 — Second `/pr-ready` run showing a clean risk report and a drafted PR title + description

![Screenshot-8](./screenshots/Second-pr-ready-clean-risk-report.png) ![Screenshot-8](./screenshots/Second-pr-ready-clean-risk-report-2.png)


---

### Notes

**1. What exactly did you change to satisfy the pre-commit hook?**

I removed the credential-shaped string that matched the secret detection pattern in the pre-commit hook and deleted the debug statement that exposed the value. Specifically, I removed the fake AWS access key (AWS_ACCESS_KEY_ID=AKIAABCDEFGHIJKLMNOP) and the line that printed it (echo "DEBUG: token is $AWS_ACCESS_KEY_ID"). After staging the updated file and attempting the commit again, the pre-commit hook no longer detected any blocked patterns, allowing the commit to succeed successfully.

---

# Task 6 — Push and Open a Pull Request Using the AI Draft

## Goal

Push your branch and open a real Pull Request, using `/pr-ready`'s drafted title and description as your starting point — read it critically and edit before you use it.

**Important:** Open this Pull Request with base repository set to **your own fork** — not the shared upstream `pravinmishraaws/devops-micro-internship-pravinmishra` repository. This assignment's hook and skill files are your own practice work, not a change meant for the shared class repo.

### Evidence

#### Screenshot 9 — Your Pull Request showing the base repository is your own fork, plus the title and description, with the `/pr-ready` draft visible for comparison (paste it in the PR conversation or your notes below)

![Screenshot-9](./screenshots/Git-pull-request-feature-ai.png)


---

#### PR Link

`https://github.com/G-abdul/devops-micro-internship-interviews/pull/2`

---

### Notes

**1. What, if anything, did you edit in the AI's drafted PR description before using it? Why?**


I expanded the AI's draft to accurately reflect all the changes included in the Pull Request. The original draft mainly described the notification script, but it did not mention the tracked hooks/pre-commit hook, the /pr-ready Claude Code skill, or the validation process. I updated the title and description to include these components so the Pull Request accurately represented the full scope of the work.

---

**2. If you had blindly copy-pasted the AI's draft without reading it, what could go wrong?**

The Pull Request description could have been incomplete or inaccurate, making it harder for reviewers to understand what actually changed. In this assignment, the AI omitted some important deliverables, such as the pre-commit hook and the shared hooks configuration. This demonstrates why AI should be used to assist with documentation, while a human remains responsible for reviewing and correcting the final content before publishing.

---

**3. Why does this PR need to target your own fork instead of the shared upstream repository?**

This assignment is intended to demonstrate my own implementation rather than contribute changes to the shared upstream repository. Opening the Pull Request against my own fork keeps the practice work separate from the original project, avoids affecting other contributors, and allows me to safely demonstrate the complete Git workflow without modifying the shared repository.

---

# Task 7 — Map the Workflow to the Agentic Loop

## Goal

Explain this assignment's workflow using the same Gather → Analyze → Human Act → Verify structure from Week 3.

### Notes

**1. Which step(s) represent Gather?**

The Gather phase included collecting information about the staged changes using the pre-commit hook and the /pr-ready skill. The hook inspected the staged files for credential-shaped strings and oversized files, while /pr-ready used git diff --cached and git status to gather information about the staged changes before analysis.

---

**2. Which step(s) represent Analyze?**

The Analyze phase occurred when the pre-commit hook evaluated the staged changes against its predefined security rules and when /pr-ready reviewed the same changes, identified potential risks, and generated a draft Pull Request title and description. Together, they provided both rule-based validation and AI-assisted review.

---

**3. Which step is Human Act, and why must a human — not Claude — run `git commit`, `git push`, and open the PR?**

The Human Act phase involved removing the fake AWS key and debug statement, staging the corrected files, committing the changes, pushing the feature branch, and opening the Pull Request. These actions change the repository's history and affect remote resources, so they require human approval and accountability. Claude provides recommendations, but it should never make these decisions or perform Git operations on its own.

---

**4. Which step is Verify?**

The Verify phase happened after the fixes were made. I successfully committed the updated file without the pre-commit hook blocking it, then ran /pr-ready again and received a clean risk report. Finally, I pushed the branch and confirmed that the Pull Request accurately reflected the intended changes.

---

**5. In one or two sentences: why do you need *both* the fixed-rule pre-commit hook and the AI skill? Isn't one enough?**

No. The pre-commit hook enforces strict, predefined security rules by blocking known risks before a commit is created, while /pr-ready provides contextual review and identifies issues that fixed rules may miss. Using both creates a stronger safety process by combining automated enforcement with intelligent analysis and human decision-making.

---

# Task 8 — LinkedIn Post

## Goal

Publish a LinkedIn post summarizing what you built and what you learned about combining fixed-rule safety checks with AI-assisted review.

### Evidence

#### LinkedIn Post URL

`https://www.linkedin.com/posts/abdulganiyu0_dmibypravinmishra-devops-git-ugcPost-7488002740275707905-t3Vw/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFamVAYBbC0P-4_t5y56JbVGUfZFmuyqJnY`

---

## Key Learnings

Add 3-5 bullet points on what you learned this week.

-Learned how to build and share a Git pre-commit hook to automatically block commits containing credential-shaped strings and oversized files before they reach the repository.

-Gained hands-on experience creating an AI-assisted /pr-ready skill that reviews staged changes, identifies potential risks, and drafts Pull Request details without modifying code.

-Strengthened my understanding of the Agentic AI workflow (Gather → Analyze → Human Act → Verify), where AI provides guidance while humans remain responsible for all Git actions.

-Learned why combining fixed-rule automation with AI-assisted review creates a stronger and more reliable code review process than relying on either approach alone.

-Improved my GitHub collaboration skills by working with feature branches, Git hooks, and Pull Requests while following secure DevOps best practices.

---

# Submission Instructions

- Ensure `hooks/pre-commit` and `.claude/skills/pr-ready/SKILL.md` are committed to your GitHub repository
- Add all required screenshots to your submission
- All written answers must be in your own words
- Do not use a real secret or credential anywhere in your submission — the fake key in Task 1 is intentional and must stay clearly fake
- Open your Pull Request against your own fork, not the shared upstream repository
- Push your final changes to your forked repository
- Include your PR link and LinkedIn post URL

---

## GitHub Repository URL

Paste your forked repository URL here:

`https://www.linkedin.com/posts/abdulganiyu0_dmibypravinmishra-devops-git-ugcPost-7488002740275707905-t3Vw/?utm_source=share&utm_medium=member_desktop&rcm=ACoAAFamVAYBbC0P-4_t5y56JbVGUfZFmuyqJnY`

---

# Completion Checklist

- [✅] Branch `feature/ai-pr-ready` created with a staged file containing a fake secret and a debug statement
- [✅] `hooks/pre-commit` created and tracked in the repo (not only in `.git/hooks/`)
- [✅] `core.hooksPath` configured to point at `hooks/`
- [✅] Pre-commit hook shown blocking the risky commit
- [✅] `.claude/skills/pr-ready/SKILL.md` created with correct `allowed-tools` (no `Write`) and `disable-model-invocation: true`
- [✅] `/pr-ready` run against the risky diff and shown flagging issues
- [✅] Risky file fixed; `git commit` succeeds cleanly
- [✅] `/pr-ready` re-run showing a clean report and drafted PR title/description
- [✅] Pull Request opened using the AI draft as a starting point, with your own fork as the base repository (not upstream), PR link included
- [✅] Agentic Loop mapping (Task 7) completed in your own words
- [✅] LinkedIn post published and URL submitted
- [✅] All required screenshots added
- [✅] GitHub repository URL provided

---

## 📌 About DMI & CloudAdvisory

DevOps Micro Internship (DMI) is a project-based DevOps program run by Pravin Mishra (The CloudAdvisory) focused on real-world execution, systems thinking, and career readiness.

It helps learners build strong DevOps foundations with hands-on experience.

---

## 📌 Resources

- 🌐 DMI Official Website: https://pravinmishra.com/dmi  
- 🎓 DevOps for Beginners (Udemy): https://www.udemy.com/course/devops-for-beginners-docker-k8s-cloud-cicd-4-projects/  
- 🎓 Agentic AI DevOps with Claude Code: https://www.udemy.com/course/ultimate-agentic-ai-devops-with-claude-code/  
- 🎓 DevOps with Claude Code: Terraform, EKS, ArgoCD & Helm: https://www.udemy.com/course/devops-with-claude-code-terraform-eks-argocd-helm/  
- ▶️ YouTube Playlist: https://www.youtube.com/playlist?list=PLFeSNDtI4Cho  
- 🔗 Pravin Mishra (LinkedIn): https://www.linkedin.com/in/pravin-mishra-aws-trainer/  
- 🏢 CloudAdvisory (LinkedIn): https://www.linkedin.com/company/thecloudadvisory/

---

*This submission is part of DevOps Micro Internship (DMI) Cohort 3 — Agentic AI Track.*
