# Data Science MBZUAI

Shared resources for data science work with AI coding agents.

## Skills

| Skill | What it does |
|---|---|
| [`my-data-science`](skills/my-data-science/SKILL.md) | A 10-phase roadmap that takes any data science problem from defining the decision to the stakeholder dashboard and monitoring. Each phase has steps, the questions to ask, checklists, decision rules, cost-based metrics and exit gates. It contains no code, so it works with any tool or domain. |

The roadmap phases are:
1. Frame the decision
2. Audit the data
3. Prepare the data and features
4. Exploratory data analysis (EDA)
5. Design the evaluation
6. Model
7. Validate and diagnose
8. Turn predictions into decisions
9. Communicate and build the dashboard
10. Deploy, monitor and iterate

## Install

### Claude Code (terminal, VS Code, desktop app)

Clone the repo and copy the skill into your personal skills folder.

macOS / Linux / WSL:

```bash
git clone https://github.com/amerbarhoush-dot/data-science-mbzuai.git
mkdir -p ~/.claude/skills
cp -r data-science-mbzuai/skills/my-data-science ~/.claude/skills/
```

Windows (PowerShell):

```powershell
git clone https://github.com/amerbarhoush-dot/data-science-mbzuai.git
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse data-science-mbzuai\skills\my-data-science "$HOME\.claude\skills\"
```

To use the skill in a single project only, copy it into that project's `.claude/skills/` folder instead. Restart Claude Code afterwards.

### Claude.ai / Claude desktop chat

1. Download the repo as a ZIP: **Code → Download ZIP**.
2. Zip **only** the `skills/my-data-science` folder, so the zip contains `my-data-science/SKILL.md`.
3. Go to **Settings → Capabilities → Skills → Upload skill** and upload that zip.

## Using it

The skill loads automatically when you ask for data-science work, such as EDA, a churn or fraud model, forecasting, anomaly detection, recommendations or a stakeholder dashboard. You can also ask for it by name: *"use the my-data-science skill"*.

## Updating

Pull the latest version and copy the folder again:

```bash
cd data-science-mbzuai && git pull
cp -r skills/my-data-science ~/.claude/skills/
```
