These reusable workflows can be used across organization's repos to extend the Github Advanced Security Features.


## Details

### Pull Requests - Enforcing No New Vulnerabilities Added  ( gas-dependency-review.yml )

By implementing this ‘check’ and requiring it to pass on all Pull Requests, engineers must not commit any code changes that include known vulnerabilities.
Out of the box, Github Advanced Security supports this feature, and requires Admin configuration to require the checks on specific criteria.

### Pull Requests - Enforcing No Aging Vulnerabilities ( gas-enforce-existing.yml )

By developing this custom check; we are able to search the repository’s **existing** packages and libraries to discover any newly published CVEs. We then filter them by age (according to a policy) and block any pull requests from being merged until the vulnerability is resolved. 

#### Default "policy"

| Severity | Max Age Allowed |
|----------|----------------|
| Critical | 2 days        |
| High     | 10 days        |
| Moderate | 60 days        |
| Low      | 90 days        |

**NOTE:  Enforcing No Aging Vulnerabilities** 

- The "policy" is baked into the workflow, effectively the aging of each severity - feel free to adjust for your own purposes
- REQUIRE this status check to pass into main
- default branch ( almost always ) is set to develop
- GAS doesnt allow filtering dependencies on branches yet
- so GAS dependencies are based off default branch (develop)

therefore.

- we let the PRs into develop but report on aging, which should be part of PR review..
- these PRs potentially could include fixes on aging vulns
- when going to production ( main ) we enforce this check to block aging vulns from going out
- *recommended* that branch-rules are implemented to enforce these checks resulting in more secure source code.
  -  e.g. `if aging-dependency-review == failure ` do not allow merge to `main`



Example usage ( caller workflow ) 

- In this example, the workflows are hosted in the org at `<YOUR_ORG>/workflows`

```
name: 'Github Advanced Security'
on:
  push:
    branches: ['develop', 'main']
  pull_request:
    # The branches below must be a subset of the branches above
    branches: ['develop', 'main']
  schedule:
    - cron: '31 3 * * 1'

# provide the appropriate permissions to the GITHUB_TOKEN
permissions:
  contents: read
  pull-requests: write
  actions: read
  security-events: write
  checks: write

jobs:
  dependency-review:
    name: Dependency Review
    if: github.event_name == 'pull_request'
    uses: <YOUR_ORG>/workflows/.github/workflows/gas-dependency-review.yml@main

  static-code-analysis:
    name: CodeQL - Static Code Analysis
    uses: <YOUR_ORG>/workflows/.github/workflows/gas-codeql.yml@main
    with:
      language: 'python'

  aging-dependency-review:
    name: Aging Dependency Review
    if: github.event_name == 'pull_request'
    uses: <YOUR_ORG>/workflows/.github/workflows/gas-enforce-existing.yml@main
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
