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

Moves issues through a project's `Status` field. The project needs a single-select `Status` field with options named: Backlog, Ready for Dev, Dev In Progress, Ready for FR, FR In Progress, Ready for Code Review, Code Review In Progress, Ready for Integration, Integration In Progress, Ready for Test, Test In Progress, Done, Cancelled, Blocked.

| Trigger | Result |
|---|---|
| Issue comment `/status <status>` | moves the issue to that status |
| PR opened | linked issues -> Ready for Code Review |
| PR comment `/code-review pass` | linked issues -> Ready for Integration |
| PR comment `/code-review fail` | linked issues -> Ready for Dev |
| PR comment `/code-review dispute` | no change (reacts with eyes) |
| PR merged | linked issues -> Integration In Progress |
| Deploy workflow | call `set-status.yml` with -> Ready for Test |

Linked issues are the ones the PR closes (`Closes #N` in the PR body). Commands only work from OWNER/MEMBER/COLLABORATOR comments. Invalid moves (for example Ready for Dev -> Done) fail the run and comment the reason on the issue; Cancelled and Blocked are allowed from any status, and any status can be entered from Blocked. Allowed moves follow the workflow diagram and are listed in `set-status.yml`.

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
