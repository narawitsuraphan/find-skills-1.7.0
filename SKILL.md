name: find-skills
slug: find-skills
displayName: Skill Discovery · Find Skills
description: Not sure which skills to install? Want the AI to automatically find the most suitable skills? "Skill Discovery · Find Skills" is a scenario-driven and keyword dual-mode skill discovery tool (skill discovery / skill finder / install skills). Whether using natural language to describe scenarios/needs (e.g., "I want to create a poster," "Help me analyze stocks," "Find a coupon/deal scraper skill"), or explicitly asking to "Install skill / find skills / find a skill / is there a skill for XX," it automatically conducts a 6-layer federated search across Official Built-in, Local Installed, SkillHub, Xiaping, GitHub, and ClawHub to recommend the best matching skills, supporting one-click installation. It completely replaces the official legacy find-skills plugin. Trigger keywords: find skills, install skills, skill recommendation, skill discovery, find skills, skill discovery, install skill.
version: "1.7.0"

find-skills (Scenario-Driven Skill Matcher)

Overview

This skill functions as a scenario-driven skill discovery engine—users describe their requirements in natural language, and the system automatically interprets the intent, conducts a federated search, and recommends the most suitable skills.

Relationship with the official legacy find-skills plugin:

The description of the official legacy find-skills plugin has been marked as DEPRECATED and will not trigger automatically.

This skill fully encompasses all its capabilities while adding official built-in scanning, local installed scanning, scenario semantic understanding, and structured recommendation rationale outputs.

Core Workflow

Step 1: Understand User Scenario

Extract the following from the user's natural language input:

Task Intent: What does the user want to accomplish?

Domain Tag: What category/field does it belong to?

Search Keywords: Both Chinese and English (for remote searching).

Examples:

"I want to design a poster" $\rightarrow$ Intent: Design/Graphics; Domain: Content Creation; Keywords: poster, design, 海报, 设计

"Analyze today's broader market trends" $\rightarrow$ Intent: Stock Analysis; Domain: Finance; Keywords: stock, A-share, market index, analysis, 大盘, 股票

Step 2: Three-Tier Federated Search

2.1 First Tier: Official Built-in Skills (Included with WorkBuddy, no installation required)

Scan the official built-in skills directory and read the name and description of each skill:

ls /Applications/WorkBuddy.app/Contents/Resources/app.asar.unpacked/resources/builtin-skills/


Use the Read tool to inspect each skill's SKILL.md YAML frontmatter and perform semantic matching against the user's scenario.

Matching Rules (by priority):

User scenario keywords appear directly in the skill's description $\rightarrow$ High score

Skill name is strongly correlated with user intent $\rightarrow$ High score

Skill description relates to the user's domain $\rightarrow$ Medium score

2.2 Second Tier: Locally Installed Skills

Scan installed skills in the following two directories:

# User-level skills
ls ~/.workbuddy/skills/

# Project-level skills (current workspace)
ls .workbuddy/skills/ 2>/dev/null || echo "No project-level skills found"


Use the Read tool to check each skill's SKILL.md YAML frontmatter, extract the name and description, and perform semantic matching with the user's scenario.

Note: If a locally installed skill also appears in Step 2.1 (Official Built-in), deduplicate and keep only the entry with the highest priority.

2.3 Third Tier: Remote Skill Marketplaces (Completely replaces legacy find-skills capabilities)

Check local marketplace cache first (Legacy find-skills Step 5 logic):

ls ~/.workbuddy/skills-marketplace/skills 2>/dev/null


If a skill matching the user's need exists in the local cache, copy and install directly without remote downloading:

cp -r ~/.workbuddy/skills-marketplace/skills/<skill-folder-name> ~/.workbuddy/skills/<skill-folder-name>


Remote Search (Legacy find-skills Step 2/2b logic):

If no results are found in the local cache, search remote marketplaces in the following order:

① SkillHub Official Marketplace (Primary source, highest priority):

curl -s "https://lightmake.site/api/v1/search?q=<URL-encoded Chinese keywords>&limit=10"
curl -s "https://lightmake.site/api/v1/search?q=<URL-encoded English keywords>&limit=10"


Filter out low-relevance results where score < 0.05.

② Xiaping Skill Marketplace (Key source for Chinese skills):

Xiaping API Base URL: https://xiaping.coze.com (Note: not coze.site)

Search Skills:

# Search skills by keywords
curl -s "https://xiaping.coze.com/api/skills/search?q=<URL-encoded keywords>&limit=10"

# Fetch skill details
curl -s "https://xiaping.coze.com/api/skills/<skill-id>"

# Fetch skill download URL
curl -s "https://xiaping.coze.com/api/skills/<skill-id>/download"


Prerequisite: Load the Xiaping Guide skill first (Path: ~/.workbuddy/skills/虾评指南/SKILL.md), and execute search and download following §1.1 + §1.2 specifications.

Important: Xiaping is the primary repository for Chinese skills. For Chinese prompts and contexts, Xiaping results are often more relevant than SkillHub.

③ GitHub Skill Repositories (Open-source skill source):

Search GitHub for repositories containing WorkBuddy / Claw skills.

Authentication Note: GitHub Search API requires authentication. If the environment variable GITHUB_TOKEN is set, requests are automatically authenticated (5,000 requests/hour); otherwise, anonymous access is used (60 requests/hour, which can easily be exhausted).

# If GITHUB_TOKEN exists, attach authentication header
if [ -n "$GITHUB_TOKEN" ]; then
  AUTH_HEADER="-H \"Authorization: Bearer $GITHUB_TOKEN\""
else
  AUTH_HEADER=""
fi

# Search code containing SKILL.md
curl -s $AUTH_HEADER "https://api.github.com/search/code?q=filename:SKILL.md+<keywords>&per_page=10"

# Search skill-related repositories
curl -s $AUTH_HEADER "https://api.github.com/search/repositories?q=<URL-encoded keywords>+skill+in:name,description&per_page=10"


Note: GitHub Code Search API (filename:SKILL.md) strictly requires authentication. If GITHUB_TOKEN is missing, fallback to searching repositories only (anonymous search supported).

Installation Method:

# Clone the skill repository from GitHub
git clone "https://github.com/<user>/<repo>.git" <target-skills-dir>/<skill-name>/

# Verify installation
ls <target-skills-dir>/<skill-name>/SKILL.md


Note: GitHub search results require manual verification to confirm valid skill structure (presence of SKILL.md), as generic code searches may return unrelated files.
