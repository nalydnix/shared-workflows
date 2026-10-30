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
      project-token: ${{ secrets.ADD_TO_PROJECT_PAT }}
```
