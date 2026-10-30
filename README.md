# shared-workflows
Reusable GitHub Actions workflows and shared automation for all my repositories.

## Workflows

### add-to-project

Adds new issues to a GitHub Project (v2). Needs a PAT or GitHub App token with project write access, stored as a secret (`GITHUB_TOKEN` cannot write to user/org Projects).

```yaml
name: Add to project
on:
  issues:
    types: [opened]
jobs:
  add:
    uses: nalydnix/shared-workflows/.github/workflows/add-to-project.yml@main
    with:
      project-url: https://github.com/users/nalydnix/projects/1
    secrets:
      project-token: ${{ secrets.GH_PAT }}
```

### Issue status automation

Moves issues through a project's `Status` field, matching status names case-insensitively. The handlers use these names, so the project's `Status` options must include: Ready for Code Review, Ready for Integration, Integration In Progress, Ready for Dev (plus whatever statuses the orchestrator sets with `/status`).

| Trigger | Result |
|---|---|
| Issue comment `/status <status>` | moves the issue to that status |
| PR opened | linked issues -> Ready for Code Review |
| PR comment `/code-review pass` | linked issues -> Ready for Integration |
| PR comment `/code-review fail` | linked issues -> Ready for Dev |
| PR comment `/code-review dispute` | no change (reacts with eyes) |
| PR merged | linked issues -> Integration In Progress |
| Deploy workflow | call `set-status.yml` with -> Ready for Test |

Linked issues are the ones the PR closes (`Closes #N` in the PR body). Commands only work from OWNER/MEMBER/COLLABORATOR comments. Any status can move to any other; the orchestrator decides which moves are valid. An unknown status fails the run and comments on the issue.

Caller workflow for each repo:

```yaml
name: Status automation
on:
  issue_comment:
    types: [created]
  pull_request:
    types: [opened, closed]
jobs:
  issue:
    if: github.event_name == 'issue_comment' && !github.event.issue.pull_request
    uses: nalydnix/shared-workflows/.github/workflows/issue-handler.yml@main
    with:
      project-url: https://github.com/users/nalydnix/projects/1
    secrets:
      project-token: ${{ secrets.GH_PAT }}
  pr:
    if: github.event_name == 'pull_request' || github.event.issue.pull_request
    uses: nalydnix/shared-workflows/.github/workflows/pr-handler.yml@main
    with:
      project-url: https://github.com/users/nalydnix/projects/1
    secrets:
      project-token: ${{ secrets.GH_PAT }}
```

Deploy step (after a successful deploy), one call per issue:

```yaml
  mark-ready-for-test:
    needs: deploy
    uses: nalydnix/shared-workflows/.github/workflows/set-status.yml@main
    with:
      project-url: https://github.com/users/nalydnix/projects/1
      issue-number: "123"
      status: Ready for Test
    secrets:
      project-token: ${{ secrets.GH_PAT }}
```
