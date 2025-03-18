# Rootstock Documentation Style Guide

This repository contains the documentation and the Vale rules for the Rootstock documentation style guide.

The style guide itself is written in Markdown and contained in the `en` directory.

It is based on:
- [Canonical Style Guide](https://github.com/canonical/praecepta)
- [Google Developer Documentation Style Guide](https://developers.google.com/style)

## Vale rules

The Vale linter operates from a series of rules. These are defined in individual YAML files, grouped into 'Styles'.
This repository contains the Rootstock set of rules. 

There are two ways to run the linting tool containing the rules.
## Set up Locally

You can setup Vale locally using the CLI or the VSCode Extension.

> To run the documentation and test locally, see [Running locally](#running-locally)

## Set up an Automation Pipeline
To set up an automated pipeline that will run on each PR. See the [Rootstock Style GitHub action](#the-rootstock-style-github-action).

## Running the Project Locally
To review the documentation locally using the Vale rules created, follow the steps:

Install Vale using the [CLI](https://vale.sh/docs/install), and integrate on VSCode to enable errors highlighting directly on the editor.

For both ways, you need to install Vale and clone the Documentation Styles Guide repository.

Open a new terminal and follow the next steps:

1. **Install Vale**

    ```shell
    brew install vale
    ```

2. **Clone the repository**

Clone the `docs-style-guide` repository. 

Note: This set of instructions will clone the repository to your home directory, and set it as a reference for the vale rules.


   ```shell
   cd ~
   git clone https://github.com/rsksmart/docs-style-guide.git
   ```

3. **Set environment variables**

Set the environment variables in your terminal, so Vale can locate the config files needed to run.

   ```shell
   export VALE_CONFIG_PATH=~/docs-style-guide/vale.ini
   export VALE_STYLES_PATH=~/docs-style-guide/styles
   ```

Note: Setting the environment variables using the method makes them available for the current terminal session only, if the terminal is closed, the variables no longer exist. You would need to add these variables to your `.bash_profile` file or (`.bashrc`, `.zshrc`), depending on your environment.

   ```shell
    echo "export VALE_CONFIG_PATH=~/docs-style-guide/vale.ini" >> .bash_profile
    echo "export VALE_STYLES_PATH=~/docs-style-guide/styles" >> .bash_profile
   ```

Close and reopen terminal to load the environment variables.

4. **Check that the rules have been loaded**

This step is optional, but you can use built-in commands to check that the configuration has been found to access the current paths:

```shell
vale ls-config
vale ls-vars
```

5. **Check indidual files or directories using the CLI**

Now that Vale is installed you can check individual files or directories locally using the vale CLI:

```shell
vale docs
vale docs/*.md
vale docs/test.md
```

6. **VSCode extension**

If you use VSCode, you can install the Vale VSCode extension, and it will automatically highlight errors and suggestions on the editor in real time.

To install it, follow these steps:

1. Install  [Vale VSCode](https://marketplace.visualstudio.com/items?itemName=chrischinchilla.vale-vscode) extension
2. Go to Settings > Vale > Vale CLI: Config > Add the absolute path to the Vale config file, located in the `docs-style-guide` directory that you cloned. Note this is the absolute path, so it will look like this: `/Users/<your-user>/docs-style-guide/vale.ini`



## 2. Run on GitHub Actions

This repository also includes a file, `action.yml`, which is the basis of a GitHub Action to automatically run Vale checks on incoming pull requests. The workflow will run linting only on the diff of the documentation files included in the PR, and add comments with the defects found.

### Using the GitHub action in a workflow

Your repository can make use of the action in a workflow. An example workflow is included in this repository and is demonstrated here. Note that the style guide action is merely part of a functioning workflow.

```yaml
name: Lint Documentation

on:
  [pull_request]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Fetch docs-style-guide repository
        run: |
          git clone https://github.com/rsksmart/docs-style-guide.git ./docs-style-guide

      - name: Install Vale via errata-ai/vale-action
        uses: errata-ai/vale-action@v2
        with:
            filter_mode: diff_context
            vale_flags: "--no-exit --minAlertLevel=error"
            reporter: github-pr-review
            fail_on_error: true
```

In the example above, the workflow is organised as a single job. This is important as the actions rely on persistence through the run.

There are three job steps:

 - The github/checkout action: this fetches the code from the repo calling the workflow
 - The style guide action: this fetches the styles and, if not present, a default config for Vale. The example fetches from the main branch since rules are currently under active development.
 - The vale/reviewdog action: this runs Vale using reviewdog, to insert comments into a pull-request

 This workflow uses reviewdog to insert output into review comments on any changes. The advantage of this method is:

  - the actions only run on new material (i.e. stuff added in the current PR)
  - it is surfaced directly where it will be noticed

[Vale styles]: https://vale.sh/docs/topics/styles/

## Customising the rules

The published Vale GitHub action ignores suggestions and turns off some features which are likely to cause a lot of noise if run in their default state. A lot of things boil down to spellings. For example, the rule which checks for sentence case in headings doesn't know that SATA is an initialism by default, so if you include it, completely reasonably, in a heading it will be marked as an error. You may wish to turn this back on **if** you have a fairly complete vocabulary list.
The example `vale.ini` file enumerates all the current rules and explicitly sets their error levels. This makes it simple to turn rules on or off, or add the spelling options, by simply making a copy of this file, editing it and including it in the repository where the checks are run. 

**Note on `vale.ini`:**

In this repository there are two vale config files:

- `vale.ini`
  - Main Configuration: Contains the actual Vale configuration and rules. This is where you define the styles, paths, and other settings for Vale.


- `.vale.ini`
  - Pointer Configuration: The .vale.ini file acts as a pointer to the main configuration file vale.ini. This is used for centralizing the configuration and making it easier to reference from different locations or repositories. 


### Adding to the rules

Anyone is welcome to submit a PR to add additional rules. However, no additions will be considered unless they are part of the Rootstock Style Guide as found in this [docs repository](https://github.com/rsksmart/devportal/blob/main/STYLE-GUIDE.md).

For a reference on rule syntax, see the Vale [documentation on Styles][Vale styles].

If you are completely new to developing Vale rules, see this [introductory guide](https://github.com/canonical/praecepta/blob/8c7fee862b2258c692439ef430198e393bdc30c4/getting-started.md). 

### Using the rules

The Vale rules are published here so that they can be used in any workflow anywhere. You can run Vale locally, as part of CI or in a GitHub workflow - all you need is Vale, a configuration file (which you can also copy from this repository) and the Rootstock Styles. Two common scenarios are also catered for more directly here, as detailed below.