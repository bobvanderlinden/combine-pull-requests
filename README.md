# Combine multiple pull requests

A GitHub action that combines multiple labelled pull requests onto the current working copy.

Not sure what to think of this? See [Use case](#Use case).

## Usage

```yml
- uses: bobvanderlinden/combine-pull-requests@v4
  with:
    label: experiment
    repo-token: ${{ secrets.GITHUB_TOKEN }}
```

## Use case

Say you have set up automatic deployment for your application in some development environment.
You work on an experimental feature for said application in a draft pull request.
The feature is not ready to be merged, but it is convenient to deploy the feature to the development environment.
Deploying your specific branch will remove all features that were on master.
Deploying your branch will also remove experimental deployments that team-mates have made.

It would be nice if we could label pull requests such that they will be picked up for deployment on top master.

This action creates a merge commit where it combines all labelled pull requests.
Building, pushing or deploying the new commit is up for other actions to solve.

An example GitHub action configuration could be the following:

```yml
name: My App CD
on:
  push:
    branches: [master]
  pull_request:
    types: [opened, labeled, unlabeled]
jobs:
  deploy:
    runs-on: ubuntu-latest
    if: ${{ github.event_name == 'push' || github.event.label.name == 'experiment' }}
    steps:
      - uses: actions/checkout@v2
        with:
          ref: master
          fetch-depth: 0
      - uses: bobvanderlinden/combine-pull-requests@v4
        with:
          label: experiment
          repo-token: ${{ secrets.GITHUB_TOKEN }}
      - run: ansible-playbook deploy.yml
```

Now every time a `experiment` label is added to a pull request a deployment is
triggered which will be based on master, but will include the labelled pull
request along with any other pull request that was labelled with `experiment`.
