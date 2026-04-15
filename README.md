<div align="center">

# Idea Generator from Supervisor and Peers

### Distill academic mentors into AI skills, generate research ideas, and discover peer groups

*A suite of Claude Code skills for AI-powered academic mentorship*

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-7c4dff.svg)](https://claude.ai/code)

</div>

---

## Overview

This repository contains three interconnected Claude Code skills that form an **academic mentor distillation ecosystem**:

```
distill-mentor  →  Generate AI mentor skills from public academic data
       ↓
scout-peers     →  Discover complementary research groups
       ↓
ideas           →  Generate novel research ideas with novelty verification
```

### Skills

| Skill | Description |
|-------|-------------|
| **[distill-mentor](distill-mentor/)** | Distill academic mentors into conversational AI skills. Auto-collect papers, analyze research style, and generate a digital mentor you can chat with. |
| **[scout-peers](scout-peers/)** | Discover research groups complementary to a mentor's frontier directions. Outputs structured data for idea generation. |
| **[ideas](ideas/)** | Generate novel research ideas based on mentor skills + latest arxiv papers, with automatic full-period novelty verification. |

---

## Quick Start

### Prerequisites

- [Claude Code CLI](https://claude.ai/code) installed
- Node.js >= 18 (for distill-mentor tools)

### Installation

```bash
# Clone the repository
git clone https://github.com/Yangming911/idea-generator-from-supervisor-and-peers.git

# Copy skills to Claude Code skills directory
cp -r idea-generator-from-supervisor-and-peers/distill-mentor ~/.claude/skills/
cp -r idea-generator-from-supervisor-and-peers/ideas ~/.claude/skills/
cp -r idea-generator-from-supervisor-and-peers/scout-peers ~/.claude/skills/

# Install dependencies for distill-mentor
cd ~/.claude/skills/distill-mentor && npm install
```

### Usage

```bash
# Step 1: Generate a mentor skill
/distill-mentor "Geoffrey Hinton" --affiliation "University of Toronto"

# Step 2: Discover complementary research groups
/scout-peers geoffrey-hinton

# Step 3: Generate research ideas
/ideas geoffrey-hinton

# (Optional) Chat with your digital mentor
/geoffrey-hinton What are the most promising research directions in deep learning?
```

---

## Research Interest Sources

The system supports two modes for determining a mentor's latest research interests:

### Mode 1: Recent Papers (Default, Open Source)

Automatically searches arxiv for the mentor's recent publications (1-2 years), clusters them by topic, and extracts trending directions. No additional configuration needed.

### Mode 2: Reading List (Local/Private)

Uses a local JSON reading list maintained by the mentor or student. Configure in the mentor profile:

```json
{
  "interest_source": {
    "type": "reading-list",
    "reading_list_path": "~/.claude/mentors/mentor-reading-list.json"
  }
}
```

This mode is useful when:
- You have access to the mentor's curated reading list
- You want to track interests that aren't reflected in published papers yet
- The mentor shares their reading list with students

---

## Project Structure

```
idea-generator-from-supervisor-and-peers/
├── distill-mentor/           # Mentor distillation skill
│   ├── SKILL.md              # Skill entry point
│   ├── prompts/              # Prompt templates
│   │   ├── analyzer.md       # Profile analysis
│   │   ├── builder.md        # Skill generation (with interest_source branching)
│   │   ├── style-analyzer.md # Research style analysis
│   │   ├── deep-paper-analyzer.md
│   │   └── public-info-analyzer.md
│   ├── tools/                # Implementation scripts
│   ├── README.md             # Detailed documentation (Chinese)
│   └── README.EN.md          # Detailed documentation (English)
├── ideas/
│   └── SKILL.md              # Research idea generator
├── scout-peers/
│   └── SKILL.md              # Peer group discovery
├── .gitignore
├── LICENSE
└── README.md                 # This file
```

---

## How It Works

```
┌─────────────────────────────────────────────────────────┐
│                  Ecosystem Workflow                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  /distill-mentor <name>                                  │
│  ├─ Multi-source search (ArXiv, Web, Scholar)            │
│  ├─ Deep paper analysis                                  │
│  ├─ Public info analysis                                 │
│  └─ Generate mentor SKILL.md + profile JSON              │
│           ↓                                              │
│  /scout-peers <mentor>                                   │
│  ├─ Extract mentor frontier directions                   │
│  ├─ Multi-source candidate discovery                     │
│  ├─ Candidate profiling                                  │
│  └─ Complementarity assessment → peers MD                │
│           ↓                                              │
│  /ideas <mentor>                                         │
│  ├─ Load mentor profile + peers data                     │
│  ├─ Frontier paper survey                                │
│  ├─ Cross-domain idea generation                         │
│  ├─ Full-period arxiv novelty verification               │
│  └─ Revised final report with rankings                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## Acknowledgements

This project is inspired by and builds upon [supervisor](https://github.com/ybq22/supervisor) by [@ybq22](https://github.com/ybq22).

---

## Contributing

We welcome contributions! Please see the individual skill READMEs for details on architecture and extension points.

### Ways to contribute

1. **Test and report** - Try the skills and file issues
2. **Add data sources** - Google Scholar, DBLP, Semantic Scholar integrations
3. **Improve prompts** - Enhance analysis quality
4. **Multi-language support** - Better support for non-English mentors

---

## Disclaimer

- Generated mentor skills are **for learning and research only**
- All information comes from **public channels** (papers, websites, interviews)
- AI-generated content is **for reference only** and does not represent the mentor's actual views
- Do not rely on generated skills for important academic or career decisions

---

## License

MIT License - See [LICENSE](LICENSE) for details.

---

<div align="center">

**Made with Claude Code & Human Collaboration**

[Get Started](https://github.com/Yangming911/idea-generator-from-supervisor-and-peers) | [Report Issues](https://github.com/Yangming911/idea-generator-from-supervisor-and-peers/issues) | [Upstream Project](https://github.com/ybq22/supervisor)

</div>
