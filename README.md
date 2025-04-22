These reusable workflows can be used across organization's repos to extend the Github Advanced Security Features.



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
  aging-dependency-review:
    name: Aging Dependency Review
    if: github.event_name == 'pull_request'
    uses: <YOUR_ORG>/workflows/.github/workflows/gas-enforce-existing.yml@main
    secrets:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
