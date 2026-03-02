---
title: "Using CI/CD to Build and Deploy a LaTeX Document"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

- How can we use CI/CD to automate the process of building and deploying a LaTeX document?
- What are the benefits of using CI/CD for this kind of task?
- How can we run LaTeX commands in our CI/CD pipeline?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Create a CI/CD pipeline that builds a LaTeX document from a source file and deploys the output
  PDF to a static URL.

::::::::::::::::::::::::::::::::::::::::::::::::

## Opening up CI/CD: Docker Images

In the previous episode we created a CI/CD pipeline that ran a few stages and a few jobs, but those
jobs didn't actually do anything, they just printed out text to the console. You may have tinkered
around with the jobs a bit and found that there were some commands that worked in the pipeline,
like `echo` and `ls`, but when you tried to run a command like `git --version` the job fails with
an error telling us that the command is not found.

This is because each job in the CI/CD pipeline runs in a Docker container, and that container only
contains a limited set of software. The upside of this is that there are a huge variety of Docker
images available that contain different software, and we just have to tell our jobs which image we
want to use.

::: callout

Docker images are like pre-built, software only operating systems. You can think of it like a small
virtual machine that comes pre-built with a specific set of software. When you run a job in CI/CD,
it starts up this virtual machine, runs the commands in the job, and then shuts down.

:::

## Selecting a different Docker Image

The default Docker image that CI/CD uses is `ubuntu:latest`, which is a very basic image that only
contains the Ubuntu operating system and a few basic utilities. If we want to run commands that
aren't found in that image, we can select a different image that does.

In the last exercise in the previous episode, we added a job that tried to run `git --version`, but
it failed because the `git` command wasn't found. We can tell CI/CD that, for this job, we want to
use a different docker image:

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  image:
    name: alpine/git:latest # Use a simple git comtainer using alpine linux
    entrypoint: [""] # Override the default entrypoint to allow us to run arbitrary commands
  script:
    - echo "Compiling the code..."
    - echo "Compile complete."
    - ls -a
    - echo "The commit SHA is ${CI_COMMIT_SHA}."
    - git --version
```

::: callout

The image is called "alpine/git:latest". The first part, "alpine/git", is the name of the image and
the second part, "latest", is the tag, which specifies the version of the image to use. If we use
the "latest" tag, it will always pull the most recent version of the image from Docker Hub. If we
use a specific tag, e.g. "alpine/git:2.52.0", it will pull that specific version of the image.

Pulling a specific version of the image is called "pinning", and is generally recommended for CI/CD
pipelines, because it ensures that your pipeline will always run with the same version of the
software, even if a new version is released that might have breaking changes.

:::

When we check the job details for this pipeline, we can see that the job is now running a different
Docker image, and the `git --version` command works successfully:

![](fig/03-build-and-deploy/git-output.png){alt="Output of git --version command in CI/CD job"}

::: callout

The CI/CD pipeline uses Docker Hub to pull images from, so we can use the
[Docker Hub website](https://hub.docker.com/) to search for images that contain the software we
want to use. We can also create our own Docker images if we need to, but there are a lot of
pre-built images already available.

:::

## Building a Basic LaTeX Document

To start with, we need to have a LaTeX document to build. Let's create a simple LaTeX document
called `main.tex` in the root of our repository with the following content:

```latex
\documentclass{article}

\begin{document}
Hello \LaTeX!
\end{document}
```

Next, we need to tell our CI/CD pipeline to use this image, and to run the command to build our
LaTeX document:

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  image:
    name: texlive/texlive:latest # Use a TeX Live image that contains LaTeX
  script:
    - echo "Generating PDF from LaTeX source..."
    - lualatex main.tex
    - echo "PDF generation complete."
    - ls -a
```

The pipeline will run automatically after each of these commits. Checking the job details for the
build job of the latest pipeline, we can see that the `lualatex main.tex` command runs successfully
and generates not only the `main.pdf` file, but also a few auxiliary files that LaTeX generates
during the build process:

![](fig/03-build-and-deploy/lualatex-output.png){alt="Output of lualatex command in CI/CD job"}

### Artifacts

Ok, so the command to build the LaTeX document ran sucessfully, but how do we get the generated PDF
file somewhere we can look at it?

We can use the "artifacts" feature of CI/CD to specify that we want to save the generated PDF file
as an artifact of the job. This means that after the job runs, we can download the generated PDF
file from the CI/CD interface. We can specify this in our `.gitlab-ci.yml` file like this:

```yaml
build-job:       # This job runs in the build stage, which runs first.
  stage: build
  image:
    name: texlive/texlive:latest # Use a TeX Live image that contains LaTeX
  script:
    - echo "Generating PDF from LaTeX source..."
    - lualatex main.tex
    - echo "PDF generation complete."
    - ls -a
  artifacts:
    paths:
      - main.pdf
```

This tells CI/CD that we specifically want to save the `main.pdf` file as an artifact of this job.
After this job runs, you should be able to see a new section in the job

::::::::::::::::::::::::::::::::::::: keypoints

- Use `.md` files for episodes when you want static content
- Use `.Rmd` files for episodes when you need to generate output
- Run `sandpaper::check_lesson()` to identify any issues with your lesson
- Run `sandpaper::build_lesson()` to preview your lesson locally

::::::::::::::::::::::::::::::::::::::::::::::::


