# tfa-assessment-skill

A skill that assesses TFA (Test Failure Analysis) Jira tickets from
the RHOAIENG TFA Sign-Off workflow. Given a ticket key or URL, it fetches the
ticket and the Jenkins build log, reconstructs the failure timeline, locates
the relevant test and product code, and shows the assessment to the user.

## Usage

```
/tfa-assessment RHOAIENG-103140
/tfa-assessment https://redhat.atlassian.net/browse/RHOAIENG-103140
```

## Installation

### With striatum (recommended)

[striatum](https://github.com/hbelmiro/striatum) packs, pushes, and installs
skills as OCI artifacts.

From a published version — check the available tags at
<https://quay.io/repository/hbelmiro/tfa-assessment-skill?tab=tags> and
replace `999-SNAPSHOT` with the proper tag:

```bash
striatum install quay.io/hbelmiro/tfa-assessment-skill:999-SNAPSHOT --target claude
```

From source, in the repo root:

```bash
striatum install . --target claude
```

See the [striatum README](https://github.com/hbelmiro/striatum#readme) for
install options (targets, personal vs. project scope).

### Without striatum

This skill has no dependencies, so striatum is not a strong requirement.
Copy the repo contents into your skills directory, e.g. for Claude Code:

```bash
mkdir -p ~/.claude/skills/tfa-assessment
cp SKILL.md ~/.claude/skills/tfa-assessment/
```

## Setup

### 1. acli (Atlassian CLI)

Install and authenticate following the official docs:
<https://developer.atlassian.com/cloud/acli/guides/install-acli/>
<https://developer.atlassian.com/cloud/acli/guides/auth/>

Verify with `acli jira workitem view RHOAIENG-103140`. If it returns
`unauthorized`, the session expired — log in again.

### 2. Jenkins credentials

The skill fetches build logs through the `jenkins_scripts` CLI. It expects
to run from a working directory containing organization directories, with
the CLI cloned under `dbasunag/` (repo:
<https://gitlab.cee.redhat.com/dbasunag/jenkins_scripts> — see its README
for usage).

It requires [uv](https://docs.astral.sh/uv/) and two environment variables:

- `JENKINS_USER` — your **short** Jenkins username (not the email; the email
  form returns 401).
- `JENKINS_TOKEN` — a Jenkins API token. Create one following the official
  docs: <https://www.jenkins.io/doc/book/system-administration/authenticating-scripted-clients/>
  When the "Add resource permissions" dialog appears, tick the minimum
  needed:

  - **Job — Read**
  - **Run — Read**

### 3. Working directory layout

Run the skill from a directory containing these organization directories.
Clone the repos you want available locally (the skill can also read them via
`gh api` without cloning, and will ask before cloning anything missing):

```text
.
├── opendatahub-io/
│   ├── opendatahub-tests        # pytest test suites
│   └── opendatahub-operator     # DSC API, webhooks, controllers
├── red-hat-data-services/
│   └── ods-ci                   # robot framework install/deploy logic
└── dbasunag/
    └── jenkins_scripts          # Jenkins log CLI
```

## Structure

```text
├── artifact.json   # striatum Skill manifest
├── SKILL.md        # entrypoint read by the AI agent
└── README.md       # this file
```
