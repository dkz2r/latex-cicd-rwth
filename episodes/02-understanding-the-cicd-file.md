---
title: "Understanding the CI/CD File"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

- What is happening in the `.gitlab-ci.yml` file?
- How can we edit the `.gitlab-ci.yml` file to add a new job to our pipeline?
- How can we edit the `.gitlab-ci.yml` file to add a new stage to our pipeline?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Understand the structure and syntax of the `.gitlab-ci.yml` file
- Edit the `.gitlab-ci.yml` file to add a new job to our pipeline
- Edit the `.gitlab-ci.yml` file to add a new stage to our pipeline

::::::::::::::::::::::::::::::::::::::::::::::::

## The `.gitlab-ci.yml` file

The `.gitlab-ci.yml` file is written in YAML, which is a human-readable markup language. It is used
to define the commands that will be run in our CI/CD pipeline. Although intention of the syntax for
this file is to be as simple as possible, it does need to conform to a specific structure in order
for GitLab to be able to read it and run our pipeline.

::: callout

`YAML` stands for "YAML Ain't Markup Language". It is a data serialization language that is often
used for configuration files. It is designed to be easy to read and write for humans, and it is
also easy to parse for computers.

:::

### YAML syntax basics

YAML files are made up of key-value pairs. The key is a string that identifies the value, and the
value can be a string, a number, a boolean, a list, or a dictionary. Key-value pairs are separated
by a colon and a space. For example:

```yaml
key: value
```

YAML files can also contain lists, which are denoted by a dash and a space. For example:

```yaml
list:
  - item 1
  - item 2
  - item 3
```

YAML files can also contain dictionaries, which are denoted by a key followed by a colon and a
space, and then the value is indented on the next line. For example:

```yaml
dictionary:
  key1: value1
  key2: value2
  key3: value3
```

### Anatomy of a `.gitlab-ci.yml` file

The first section in our file looks like this:

```yaml
stages:          # List of stages for jobs, and their order of execution
  - build
  - test
  - deploy
```

This is defining the stages of our pipeline and the order in which they will be run.

Next, we have the definition of our first job:

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."
```

The job  name is the key (`build-job`) and the value is a dictionary that contains the details of
the job. The keys of a job are specific to GitLab - a list of available job keywords can be
found in the [GitLab documentation - Job Keywords](https://docs.gitlab.com/ci/yaml/#job-keywords).

In our case, we have two keys - `stage` tells us which stage this job belongs to, and `script` is
a list of commands that will be run when the job is executed.

Note that the next three jobs are largely the same:

```yaml
unit-test-job:   # This job runs in the test stage.
  stage: test    # It only starts when the job in the build stage completes successfully.
  script:
    - echo "Running unit tests... This will take about 60 seconds."
    - sleep 60
    - echo "Code coverage is 90%"

lint-test-job:   # This job also runs in the test stage.
  stage: test    # It can run at the same time as unit-test-job (in parallel).
  script:
    - echo "Linting code... This will take about 10 seconds."
    - sleep 10
    - echo "No lint issues found."

deploy-job:      # This job runs in the deploy stage.
  stage: deploy  # It only runs when *both* jobs in the test stage complete successfully.
  environment: production
  script:
    - echo "Deploying application..."
    - echo "Application successfully deployed."
```

We can define as many jobs as we like for each stage, but each job must have a unique name and
must belong to a stage that is defined in the `stages` section of the file. We cannot have a stage
for which there are no jobs. We also cannot have a job that belongs to a stage that is not defined
in the `stages` section.

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Add a new Job to the Pipeline

Use the pipeline editor to add a new job to the `test` stage of our pipeline. This job should be
called `my-test-job` and it should run the following commands:

```bash
echo "Running my test job... This will take about 30 seconds."
sleep 30
echo "My test job is complete."
```

:::::::::::::::::::::::: solution

```yaml
my-test-job:
  stage: test
  script:
    - echo "Running my test job... This will take about 30 seconds."
    - sleep 30
    - echo "My test job is complete."
```

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Add another Job to the Pipeline

Use the pipeline editor to add the following job to the `test` stage of our pipeline:

```yaml
validate-test-job:
  stage: validate
  script:
    - echo "Running my validate test job... This will take about 20 seconds."
    - sleep 20
    - echo "My validate test job is complete."
```

What else do we need to add to our `.gitlab-ci.yml` file in order for this job to run successfully?

:::::::::::::::::::::::: solution

We must add the `validate` stage to our `stages` section at the top of the file:

```yaml
stages:
  - build
  - test
  - validate
  - deploy
```

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 3a: Add commands to a job

Add some additional commands to the `script` section of the `build-job` job. Start with `ls -a` to
list all files in the current directory. Then add a command to print out the commit SHA of the
current commit using the `CI_COMMIT_SHA` environment variable.

Try replacing the CI_COMMIT_SHA with something else from the list of
[GitLab CI/CD Predefined Variables](https://docs.gitlab.com/ci/variables/predefined_variables/) and
see what happens.

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."
    - ls -a
    - echo "The commit SHA is ${CI_COMMIT_SHA}."
```

:::::::::::::::::::::::: solution

`ls -a` will print out all files in the current directory, including hidden files. Your output
might look something like this:

```output
$ ls -a
.
..
.git
.gitlab-ci.yml
README.md
```

Note that also present in the output is the `README.md` file - when we run a CI/CD job, the runner
will automatically clone our repository and run the commands in the context of that repository!

Depending on the variables you picked, you might find that some of them print values, and others
are empty. Some variables are only available in certain contexts, e.g. `CI_COMMIT_SHA` is only
available when the pipeline is triggered by a commit. We will see later how to add our own
variables.

:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 3b: Add commands to a job

Try adding a command for a common command line utility:

- `git --version` to get the version of git that is installed in the runner

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."
    - ls -a
    - echo "The commit SHA is ${CI_COMMIT_SHA}."
    - git --version
```

What happens? Why?

:::::::::::::::::::::::: solution

The pipeline will fail with an error message that looks something like this:

```output
$ git --version
/usr/bin/bash: line 171: git: command not found
```

This is because the runner that is executing our pipeline is using a Docker image that does not
have git installed. We will see in the next episode how to change the Docker image that our runner
uses.


:::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

- Use `.md` files for episodes when you want static content
- Use `.Rmd` files for episodes when you need to generate output
- Run `sandpaper::check_lesson()` to identify any issues with your lesson
- Run `sandpaper::build_lesson()` to preview your lesson locally

::::::::::::::::::::::::::::::::::::::::::::::::
