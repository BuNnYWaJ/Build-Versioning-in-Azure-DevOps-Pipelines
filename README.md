---
title: Build-Versioning-in-Azure-DevOps-Pipelines
---

## Introduction:

Versioning is one of the important aspects of software build and release cycles. There are different principles around versioning your changes, but the most commonly adopted system is Semantic Versioning also called SemVer.

In short semantic versioning, is denoted by X.Y.Z where X is the major version, Y is a minor version, and Z is the patch. You can read more about the semantic versioning here https://semver.org

Azure DevOps is a CI/CD platform by Microsoft. It is a pretty neat platform with a lot of out of box tools to automate your CI/CD pipelines.

The scope of this article is restricted to application builds and versioning using Azure Pipelines YAML file. We will cover this article in two iterations. The first will be versioning PR and feature branch builds, and the second will be versioning a master branch. The idea is to version the master branch differently than PR and feature branch builds.

Tools like Gitversion and Minver also help achieve semantic versioning.

Before we dig into the main topic, some introduction to Azure DevOps pipelines.

By default, Azure DevOps will version your build with YYYYMMDD.Revision format.

For example, if we build the below azure-pipelines.yml file, the first build number will be 20191207.1 and subsequent builds will be incremented as 20191207.2, 20191207.3 … and henceforth.

Now, let’s add our custom versioning, for this purpose I have declared two properties variables and name. Variables property has major and minor variables declared (You can declare these variables globally using variable groups and linking the variable group name to the pipeline)and name property has the custom versioning format.

When we build the YAML, our build will be versioned with 1.1.1 (major_version.minor_version.patch format) and subsequent builds will have 1.1.2,1.1.3,1.1.4…

## Building and versioning PR builds and feature branches
## Feature Branch Versioning:

As developers start working on feature branches and raise PR’s to merge into the main branch, we need to make sure the code quality is maintained by the code review process and successful PR builds. So we have to define a versioning system for these builds. Again, a lot of this depends on your branching strategy, this is just a versioning example of the common flow of software development.

Let’s start by creating a new feature branch from master, let’s call it feature_branch_changes. In this branch, we will add the below lines to the azure-pipelines.yml file so that our feature branch is versioned differently.

We have created a stage called Build_Branch_Version_Number, in which we have declared a condition that allows this stage to be run only if the branch is not master. Also, we have declared major and minor variables to 2 and 0.

# condition: ne(variables['Build.SourceBranch'], 'refs/heads/master')