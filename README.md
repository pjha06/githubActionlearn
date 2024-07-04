# SalesforceOneOrg

Repository used for the "One Employ" single Salesforce org

## Contents

[Local Development](#local-development)

-   [Prerequisites](#prerequisites)
-   [Cloning the Repository](#cloning-the-repository)
-   [Project setup](#project-setup)

[Creating scratch org](#creating-scratch-org)

[Branching strategy](#branching-strategy)

[Commit strategy](#commit-messages-convention)

[Pipelines](#pipelines)

-   [Pull Request Build](#pull-request-build)
-   [Scratch Org Pool](#scratch-org-pool)
-   [Deployment](#deployment)

[Testing](#testing)

[Lightning Web Components](#lightning-web-components)

[Prettier](#prettier)

[Code linting](#code-linting)

[Salesforce Code Analyzer](#salesforce-code-analyzer)

-   [PMD](#pmd)
-   [Retire JS](#retire-js)
-   [Salesforce Graph Engine](#salesforce-graph-engine)

[Lightning Flow Scanner](#lightning-flow-scanner)

[Relevant Links](#relevant-links)

## Local Development

### Prerequisites

1. Install [`Node.js`](https://nodejs.org/en).
1. Install [`Git`](https://git-scm.com/).
1. Install [`Salesforce CLI`](https://developer.salesforce.com/tools/salesforcecli)

Preferred development tool is Visual Studio Code with Salesforce Extensions. Documentation and links to install can be found in [Salesforce documentation](https://developer.salesforce.com/tools/vscode).

### Cloning the Repository

When working on a new feature, any changes made in the `develop` branch should be pulled down to ensure your local environment has the latest changes.

1. If you have not yet cloned the repository, or would like to start fresh, make sure you have [set up your SSH key for GitHub account](https://employinc.atlassian.net/l/cp/8w86t5jH), and run:

```
git clone git@github.com:EmployInc/SalesforceOneOrg.git
cd SalesforceOneOrg
```

You can also [clone git repository directly from Visual Studio Code](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git#_clone-a-repository-locally) without using command line

2. If you already have cloned the repository and have changes you would like to keep, run:

```
cd SalesforceOneOrg
git pull origin develop
```

You can also [use Visual Studio Code UI to pull the changes](https://code.visualstudio.com/docs/sourcecontrol/intro-to-git#_pushing-and-pulling-remote-changes) without using command line

### Project setup

This will set up your project for local development, and should be done each time you work on a new feature.

1. Install code dependencies:

```
npm install
```

2. Install SF CLI plugins:

```
sf plugins install @salesforce/sfdx-scanner
sf plugins install lightning-flow-scanner
```

3. Install `sfpowerscripts`:

```
npm i -g  @dxatscale/sfpowerscripts
```

4. Authorize with Production in SFDX:

```
sf org login web --set-default-dev-hub
```

You can also use Visual Studio Code and Salesforce Extensions to authorize with Production without using command line. Press Ctrl+Shift+P (Windows or Linux) or ⇧⌘P (macOS) to open Command Palette and use `SFDX: Authorize a Dev Hub`.

### Creating scratch org

#### Using Scratch Org Pool **[RECOMMENDED]**

The easiest way to get scratch org is to use `sfpowerscripts` and fetch prepared scratch org directly from the dev pool that always has 10 fresh scratch orgs available.

1. Make sure you are on the `develop` branch in the repository.
1. Call following command: `sfp pool:fetch --setdefaultusername --tag dev-pool --targetdevhubusername <PROD_USERNAME>` where `<PROD_USERNAME>` should be replaced with your username in Production org.
1. Call `sf project deploy start --wait 30` to make sure all recent changes in `develop` branch have been moved to the scratch org.

#### Creating scratch org by yourself **[NOT RECOMMENDED]**

Sometimes, if scratch org pool is not working, we may need to create scratch org manually. Following steps must be taken to create scratch org:

1. Call following command:

    `sf org create scratch --alias <SO_ALIAS> --set-default --definition-file ./config/project-scratch-def.json --target-dev-hub <PROD_USERNAME> --wait 10 --duration-days 30`

    Where `<SO_ALIAS>` should be replaced with alias for the scratch org and `<PROD_USERNAME>` should be replaced with your username in Production org.

    For example:

    `sf org create scratch --alias MyScratchOrg --set-default --definition-file ./config/project-scratch-def.json --target-dev-hub deployment_user@employinc.com --wait 10 --duration-days 30`

1. Call following command:

    `GAINSIGHT_INSTALLATION_KEY=<GAINSIGHT_KEY> CI_SCRATCH_ORG_DEPLOY=true sh ./scripts/pools/dev-pool-post.sh <SO_USERNAME>`

    Where `<SO_USERNAME>` should be replaced with username of the scratch org. You can use `sf org display user` command to get the username.

    Where `<GAINSIGHT_KEY>` should be replaced with the installation key for the Gainsight package.

    For example:

    `GAINSIGHT_INSTALLATION_KEY=XXX CI_SCRATCH_ORG_DEPLOY=true sh ./scripts/pools/dev-pool-post.sh test-qq7c9dl0ddbw@example.com`

## Branching strategy

This project uses following long-living branches:
Name | Purpose
--- | ---
develop | Base develop branch that is pointing to sandboxes
main | Release branch that is pointing to Production

During development process we have to identify type of item we are working on and then create correct branch out of `develop`:

-   `feature/EMSOC-XXX` when working on a story or task
-   `bugfix/EMSOC-XXX` when working on a bug ticket
-   `hotfix/EMSOC-XXX` when creating hotfix

After work is done, pull request to `develop` needs to be created.

## Commit messages convention

Every time you make a commit you should include a ticket number in the commit message. Sample commit messages:

-   EMSOC-1: Added Account validation rules
-   EMSOC-123 - Added Account page layout
-   EMSOC-43 Added integration with ZoomInfo

## Pipelines

### Pull Request Build

This pipeline is triggered every time a new pull request is created to `develop` branch and runs static code analysis for committed files and validates source on the scratch org. Definition of the pipeline for pull request build is stored in the [`pull-request.yml`](.github/workflows/pull-request.yml) file.

### Scratch Org Pool

#### Scheduled

This pipeline is triggered every weekday at 10 PM UTC. It is creating two pools of scratch orgs - one for the developers and one for the CI. Definition of the pipeline for scheduled scratch org pool is stored in the [`scheduled-scratch-org-pool.yml`](.github/workflows/scheduled-scratch-org-pool.yml) file.

#### Automated re-creation

Every time configuration files are changed in the repository and merged to `develop` branch, all pools will be deleted and re-created with then new configuration. Definition of the pipeline for automated scratch org pool re-creation is stored in the [`develop-push.yml`](.github/workflows/develop-push.yml) file.

#### Manual

Additionally, pipeline to create scratch org pool can be triggered manually, outside of the scheduled context by going to the Actions in GitHub and finding `Create Scratch Org Pool` action. Definition of the pipeline is stored in the [`create-scratch-org-pool.yml`](.github/workflows/create-scratch-org-pool.yml) file.

### Deployment

This pipeline runs deployment to the specified org. It supports two deployment types:

-   Delta - used when you want to deploy only delta between last release and current release
-   Full - used when you want to perform full deployment

It can be triggered manually by going to the Actions in GitHub and finding `Deploy` action. It can be also triggered by other workflows and currently every merge into `develop` branch will trigger deployment of the changes to the `Dev-Stage` sandbox. Definition of the pipeline is stored in the [`deploy.yml`](.github/workflows/deploy.yml) file.

## Testing

LWC tests:

```
npm run test:lwc
```

Apex tests:

```
npm run test:apex
```

## Lightning Web Components

When possible, UI components should be written as [Lightning Web Components](https://developer.salesforce.com/docs/platform/lwc/guide).

### Folder Structure

1. Components live in `force-app/main/default/lwc`. Components should have their own directory as a direct child of `lwc`, and no nesting beyond that (besides the `__tests__` folder).

### Testing

1. Tests should be placed in a `__tests__` directory as a direct child of the folder for the component under test. Tests should be named `{file_being_tested}.test.js`.

2. Tests can be run with

    ```
    npm run test:lwc
    ```

3. Mocks for your tests should be placed in `force-app/test/jest-mocks`, and wired up correctly in `jest.config.json`.

## Prettier

This project uses prettier to have common style guide and format SF specific files as well as standard file formats. Every developer using Visual Studio Code should install [Prettier extension](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode).

There are couple of commands available in this project for using prettier:

-   `npm run prettier` will run prettier for all of the files in the repository and update formatting.
-   `npm run prettier:verify` will run prettier for all of the files in our repository and verify if they are formatted properly.

## Code Linting

This project uses ESLint to identify stylistic errors and erroneous constructs in JavaScript files.

Every developer using Visual Studio Code should install [ESLint extension](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint).

`npm run lint` command will run eslint for all of the Aura and LWC components.

## Salesforce Code Analyzer

This project uses Salesforce Code Analyzer for source code analysis to review and improve the code.

Every developer using Visual Studio Code should install [Salesforce Code Analyzer extension](https://marketplace.visualstudio.com/items?itemName=salesforce.sfdx-code-analyzer-vscode) and review related [documentation](https://forcedotcom.github.io/sfdx-scanner/en/v3.x/code-analyzer-vs-code-extension/).

### PMD

This project uses PMD for static code analysis to find common programming flaws and enforce best practices in Apex.

`npm run pmd` command will run pmd for all Apex classes and triggers.

### Retire JS

This project uses RetireJS to identify security vulnerabilities in third-party JavaScript dependencies.

`npm run retirejs` command will run retirejs for all of the codebase.

### Salesforce Graph Engine

This project uses Salesforce Graph Engine to identify security vulnerabilities and code issues in Apex code.

`npm run dfa` command will run graph engine for all of the Apex classes.

## Lightning Flow Scanner

This project uses Lightning Flow Scanner to identify potential issues and improvements in Salesforce Flows.

Developers may want to use [Visual Studio Code extension](https://marketplace.visualstudio.com/items?itemName=ForceConfigControl.lightningflowscanner) for easier usage of the tool.

`npm run flow:scanner` command will run flow scanner for all of the Flows.
`npm run flow:scanner -- --config .flow-scanner-naming-convention.json` command will run flow scanner for all of the Flows and checks if it has correct name.

## Relevant Links

[Repository](https://github.com/EmployInc/SalesforceOneOrg)

[JIRA Project](https://employinc.atlassian.net/jira/software/projects/EMSOC/)

[Confluence Home](https://employinc.atlassian.net/wiki/spaces/EPAM/overview)

[Production Org](https://employinc.my.salesforce.com/)

[Dev-Stage Org](https://employinc--devstage.sandbox.my.salesforce.com/)

[Dev-CPQ Org](https://employinc--devcpq.sandbox.my.salesforce.com/)

[Dev-Marketo Org](https://employinc--devmarketo.sandbox.my.salesforce.com/)

[Data Migration Org](https://employinc--employdm.sandbox.my.salesforce.com/)

[Lever Org](https://employinc--lever.sandbox.my.salesforce.com/)

[QA Org](https://employinc--employqa.sandbox.my.salesforce.com/)

[UAT Org](https://employinc--employuat.sandbox.my.salesforce.com/)
