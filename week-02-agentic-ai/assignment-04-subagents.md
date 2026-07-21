# Assignment 4 — Building Your AI Team

Part of the DevOps Micro Internship (DMI) Cohort 3 with Agentic AI

---

## Purpose

In this assignment, you will build and configure a set of specialized AI subagents inside your project. You will learn how different models and tool permissions define agent behavior, and you will trigger two real agent delegations to analyze security and cost aspects of your Terraform infrastructure.

---

# Task 1 — Create the Agents Folder and Add Files

## Goal

Create the `.claude/agents/` directory and add all required agent files.

### Evidence

#### Screenshot 1 — VS Code sidebar showing `.claude/agents/` with all 3 files

![Screenshot-1](./screenshots/VS-Code-claude-agents.png)


---

# Task 2 — Compare the Agent Configurations

## Goal

Analyze the configuration differences between the three agents and demonstrate understanding of model and tool selection.

### Written Answers

#### 1. Why does the cost optimizer use Haiku instead of Sonnet?

The cost optimizer mainly analyzes configurations and suggests ways to reduce costs. It does not require deep reasoning or complex code generation for most tasks. Using Haiku makes sense because it is faster and more cost-effective while still being capable of identifying common optimization opportunities. This helps reduce AI usage costs without sacrificing much quality for the type of work being performed.

---

#### 2. Why does the security auditor NOT have Write in its tools list?

A security auditor's role is to inspect, review, and identify risks, not to make changes. Removing Write access helps prevent accidental modifications to infrastructure, configurations, or code while the audit is being performed. This follows the principle of least privilege, where a tool only receives the permissions necessary to perform its job safely.

---

#### 3. Why does the tf-writer use `inherit` instead of a specific model?

Using inherit allows the tf-writer to automatically use whichever model is currently configured for the session instead of being locked to a specific one. This provides flexibility because the same skill can benefit from future model upgrades or different model selections without requiring changes to its configuration. It also helps keep the setup simpler and easier to maintain.

---

### Evidence

#### Screenshot 2 — `security-auditor.md` frontmatter showing model and tools configuration

![Screenshot-2](./screenshots/Security-auditor.png)


---

#### Screenshot 3 — `cost-optimizer.md` frontmatter showing the model and tools configuration

![Screenshot-3](./screenshots/Cost-optimizer.png)

---

# Task 3 — Run the Security Auditor

## Goal

Trigger the security auditor agent and analyze the generated security report for your Terraform infrastructure.

### Evidence

#### Screenshot 4 — The delegation message showing Claude launched the security-auditor

![Screenshot-4](./screenshots/Security-auditor-launch.png)

---

#### Screenshot 5 — Security audit report output

![Screenshot-5](./screenshots/Security-audit-report.png) ![Screenshot-5](./screenshots/Security-audit-report-2.png)

---

# Task 4 — Run the Cost Optimizer

## Goal

Trigger the cost optimizer agent and review the generated cost optimization report.

### Evidence

#### Screenshot 6 — The full cost optimization report

![Screenshot-6](./screenshots/full-cost-optimization-report.png) ![Screenshot-6](./screenshots/full-cost-optimization-report-2.png)

---

# Submission Instructions

- Ensure all agent files are committed in `.claude/agents/`
- Complete all written answers in your GitHub Repo
- Push final changes to your forked GitHub repository

---

## GitHub Repository URL

`https://github.com/G-abdul/Ultimate-Agentic-DevOps-with-Claude-Code.git`

`________________________________________________________________________`


---

# Completion Checklist

- [✅] `.claude/agents/` folder contains all 3 agent files
- [✅] Screenshot 2 shows correct `security-auditor.md` configuration
- [✅] Screenshot 3 shows correct `cost-optimizer.md` configuration
- [✅] All 3 written answers completed 
- [✅] Security auditor executed successfully
- [✅] Cost optimizer executed successfully
- [✅] Security report is visible with findings
- [✅] Cost report is visible with recommendations
- [✅] All required screenshots added
- [✅] GitHub repo updated with agents

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