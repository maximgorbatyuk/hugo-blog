---
layout: post
title: How to increase nuget package version automatically with Github Actions
category: development
tags: [.net, nuget, ci/cd, github, tutorial]
date: "2023-02-03"
description: "When you push new nugets to nuget.org, you need to increase the version of the package. This article describes how to do it automatically with Github Actions."
---

Why do you need to increase the version of the nuget package? When you push a new nuget package to nuget.org, you need to increase the version of the package. Otherwise, you will get an error. I will show you how to do it automatically with Github Actions. As an example, I will use my nuget package [MaximGorbatyuk.DatabaseSqlEndpoints](https://github.com/maximgorbatyuk/MaximGorbatyuk.DatabaseSqlEndpoints).

## TL;DR

- You need `github.run_number` variable to increase the version,
- The [release.yml](https://github.com/maximgorbatyuk/MaximGorbatyuk.DatabaseSqlEndpoints/blob/dev/.github/workflows/release.yml) file as example for you,
- Code example - [MaximGorbatyuk.DatabaseSqlEndpoints](https://github.com/maximgorbatyuk/MaximGorbatyuk.DatabaseSqlEndpoints).

## Workflow

Usually, I use the following workflow:

1. Work on a feature in a separate branch. For example, `feature/1234-add-new-feature`.
2. Create a pull request to the `dev` branch.
3. Code in the pull request is tested by Github Actions CI.
4. If the tests are successful, the pull request is merged to the `dev` branch by me.
5. If I decide to deply new version of nuget, I merge the `dev` branch to the `main` branch.
6. New code in the `main` branch is used for building nuget package.
7. the nuget package is deployed to nuget.org.

For automatic version increase, I use a little hack. Github has run_number variable that represents the order number of the pipeline run. I use this number as last version number of the nuget package.

The main problem is how to put this number into the csproj file. Here I use a [sed](https://www.geeksforgeeks.org/sed-command-in-linux-unix-with-examples/) command to replace the version number in the csproj file inside `<Version>1.0.0</Version>`. The sed command looks like this:

```bash
sed -i "s/\(<Version>\([0-9]\+\.\)\{2\}\)\([0-9]\+\)/\1${{github.run_number}}/" ${{env.PROJECT_FILE}}
```
For example, the version is `1.0.0` and the `github.run_number` is `5`. The result will be `1.0.5`. That's what I need. Let's see how to use it in Github Actions.

## How to setup repository

First of all, you need to add two yml files to your repository:

- `test.yml`
- `release.yml`

The first one [test.yml](https://github.com/maximgorbatyuk/MaximGorbatyuk.DatabaseSqlEndpoints/blob/dev/.github/workflows/test.yml) will be responsible for testing the code. The second one [release.yml](https://github.com/maximgorbatyuk/MaximGorbatyuk.DatabaseSqlEndpoints/blob/dev/.github/workflows/release.yml) will be responsible for building the nuget package and deploying it to nuget.org.

The `test.yml` is very simple. It looks like this:

```yml
name: Build and test

on:
  push:
    branches: [dev] # run tests on push to dev branch
  pull_request:
    branches: [dev] # OR run tests on pull requests to dev branch

jobs:
  Test:
    runs-on: ubuntu-latest
    env:
      solution-file: MaximGorbatyuk.DatabaseSqlEndpoints.sln
    steps:
      - uses: actions/checkout@v2
      - name: Backend build and test
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: "6.0.x" # SDK Version to use.

      - name: Install dependencies # installing dependencies for all projects in the solution
        run: dotnet restore ${{env.solution-file}}

      - name: Test # running tests for all projects in the solution
        run: dotnet test ${{env.solution-file}} --no-restore --verbosity normal
```

The `release.yml` is a bit more complicated. It looks like this:

```yml
name: Publish nuget

on:
  push:
    branches: [main] # run deployment on push to main branch
jobs:
  build:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    env:
      project: MaximGorbatyuk.DatabaseSqlEndpoints
      PROJECT_FOLDER: ./MaximGorbatyuk.DatabaseSqlEndpoints
      PROJECT_FILE: MaximGorbatyuk.DatabaseSqlEndpoints/MaximGorbatyuk.DatabaseSqlEndpoints.csproj
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: "6.0.x" # .NET SDK version to use.

    # github.run_number is a number of the pipeline run. It is increased automatically.
    # That's why we can use it to increase the version of the nuget package.
    # Here we replace the last number in the version with github.run_number.
    - name: Set version
      run: |
       echo "Buid version is ${{github.run_number}}"
       sed -i "s/\(<Version>\([0-9]\+\.\)\{2\}\)\([0-9]\+\)/\1${{github.run_number}}/" ${{env.PROJECT_FILE}}

    - name: Build
      working-directory: ${{env.PROJECT_FOLDER}}
      run: dotnet build --configuration Release

    - name: Pack
      working-directory: ${{env.PROJECT_FOLDER}}
      run: dotnet pack --configuration Release --no-build --output .

    # Here we use external public config file to publish nuget package.
    # The script needs to know the path to the csproj file and the nuget API key.
    - name: Publish nuget package
      uses: brandedoutcast/publish-nuget@v2.5.2
      with:
          PROJECT_FILE_PATH: ${{env.PROJECT_FILE}}
          NUGET_KEY: ${{secrets.NUGET_API_KEY}}
```

The `NUGET_API_KEY` is a Github secret variable that contains the API key for nuget.org. [Here](https://learn.microsoft.com/en-us/nuget/nuget-org/publish-a-package) you can learn how to get it. Here is a screenshot of the secret variable in my repository:

![Nuget API key as secret](/images/blog/development/2023-02-03/github_secrets.png)

That's all. Now you can push your code to the `dev` branch and it will be tested by Github Actions. When you are ready to deploy new version of nuget package, you need to merge the `dev` branch to the `main` branch. After that, the `release.yml` will be triggered and the new version of the nuget package will be deployed to nuget.org.

I hope this tutorial will be useful for you. If you have any questions, you may find my contacts [here](https://maximgorbatyuk.com/about/#where-you-can-find-me).
