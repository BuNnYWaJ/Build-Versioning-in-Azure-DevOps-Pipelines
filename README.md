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

## condition: ne(variables['Build.SourceBranch'], 'refs/heads/master')

If the branch is not master, it declares a variable called brpatch which uses a counter expression.

## Counter expression in Azure DevOps: 
A counter expression increments the seed value based on its prefix value. Below, the seed value starts with 0, and the prefix value is variable minor i.e 1. Each run of the pipeline will increment its seed value. If we change the minor variable value to 2, the seed value resets to 0 as the prefix has changed. For more information visit: https://docs.microsoft.com/en-us/azure/devops/pipelines/process/expressions?view=azure-devops#counter

### syntax: $[counter(variables[‘prefix’], seed)]

### $[counter(variables[‘minor’], 0)]

In a similar way, we will use build.sourcebranchname as prefix and start our seed value with 0, so for each run of the pipeline, the value of variable brpatch will increment as 0,1,2…

variables:
       brpatch: $[counter(variables['build.sourcebranchname'], 0)]

As the next step, we will use Azure DevOps’s build.updatebuildnumber function to update the version of the running pipeline. This function will only run when the build.reason is not equal to Pull Request.

echo "##vso[build.updatebuildnumber]$(major).$(minor)-$(Build.SourceBranchName).$(brpatch)"
condition: ne(variables['Build.Reason'], 'PullRequest')

When we build the feature_branch_changes, we should see build number 2.0-feature_branch_changes.0. All subsequent builds of the feature_branch_changes branch will be incremented like 2.0-feature_branch_changes.1,2,3…n

## PR versioning

Now, let’s add versioning to PR builds in a similar way. Once we complete our changes to feature_branch_changes branch, let's create a PR against the master branch, name it pr_for_feature_branch_changes and build it. You can add a build policy to build PR’s automatically and successfully before merging to the master branch.

For pull requests, we will use system.pullrequest.pullrequestid as prefix and start our seed value with 0, so for each run of this PR , the value of variable prpatch will increment as 0,1,2…and build will be versioned as 2.0-PullRequest.0,1,2,3…n

## Building and versioning master branch

Before we merge our PR into master lets add versioning for the master branch.

Here, we have a stage named Build_Master_Version_Number, which will run only when Build.SourceBranch is master

### condition: eq(variables['Build.SourceBranch'], 'refs/heads/master')

and will version the build with the declared variable minor as prefix and 0 as seed.

### patch: $[counter(variables['minor'], 0)]

Once the build is completed after the merge, we should see version 2.0.0 as the build version for master branch and all subsequent builds will be 2.0.0, 2.0.1, 2.0.2, 3, 4….n

This is one of the ways you can version master, feature branch and PR builds in a unique way using Azure Devops.

Hope you enjoyed this article!

