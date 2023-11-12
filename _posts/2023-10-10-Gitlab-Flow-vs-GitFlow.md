---
layout: post
title: Gitlab Flow vs. GitFlow
date: 2023-10-10
categories: [Gitlab, GitFlow, Workflow, Git, CI/CD]
---
## Introduction

Recently, I delved into different branching models for a project at work. I was originally asked to look into GitFlow, but since the project already uses Gitlab, I came across a lot of Gitlab Flow information as well. This post distills what I learned from studying both methodologies

## A Deep Dive into GitFlow

[Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) revolves around multiple long-lived branches. It uses separate branches for development of new features ("feature branches"), preparation and release of versions ("release branches"), and hotfixing of production releases ("hotfix branches"). It is particularly good when your team has larger, more complex projects with formal release cycles. 

### Branching Strategy

- **Feature branches** originate from the Develop branch and are merged back in once that feature is complete.
- **Release branches** are forked from the Develop branch when gearing up for a release. These branches are used to prepare releases and apply last minute fixes. Once ready, the release branch is merged into both Develop and Master branches. The latter should be tagged with a version.
- **Hotfix branches** are created from Master to address critical bugs in production and are merged into both Develop and Master branches.
- **Develop and Master branches**
  - Master stores the official release history.
  - Develop branch is where features are integrated and tested before merging into master, and stores the full history.

### Advantages

- Clear separation of features, releases, and hotfixes.
- Structured approach, making it suitable for projects with scheduled releases.

### Challenges

- Can be complex to manage, especially for smaller teams or projects.
- Might be overkill for projects that deploy frequently.
- Can be difficult to use with CI/CD.

## Unpacking GitLab Flow

In [Gitlab Flow](https://about.gitlab.com/topics/version-control/what-are-gitlab-flow-best-practices/), the Master branch always reflects a production-ready state. Features seamlessly merge into Master after thorough testing and reviews. While `release` and `hotfix` branches are optional, they can prove invaluable for staging and immediate fixes. The resultant workflow is streamlined, making it ideal for continuous delivery.

GitLab further endorses this approach with inbuilt features such as Releases and Environments. An integrated issue tracker enhances the synergy between issues and merge requests.

### Branching Strategy

- **Feature branches** are created from master branch and merged back into master once the feature is complete.
- **Master branch** as the main integration branch, and always reflects a production-ready state.
- **Production or release branches**: Used for staging releases before a master deployment.
  - **Releases** are Gitlab objects that created code artifacts such as Docker images, Java JARs, etc. They can have helpful links to Runbooks or other useful documentation that can help during deployments.
  - **Environments** are another Gitlab object that represent deployment targets. Each environment branch can track the state of the codebase in a particular environment (staging, production etc).
    - Environments allow associating merge requests and pipelines with specific deployment targets. The status of an environment (e.g. passed/failed/running) provides visibility into the deployability of code.
- **Environment** branches for specific deployment targets (if used).

### Advantages

- Emphasis on simplicity with an unwavering focus on continuous delivery/integration (CI/CD).
- Adaptable to a spectrum of project requirements.
- Synergizes well with trunk-based development paradigms.

### Challenges

- Requires robust CI/CD practices in place.
- Less structured compared to GitFlow, which can be a challenge for projects that need strict release cycles.

## Which One to Choose?

- **For Git Novices**: GitLab Flow, with its intuitive nature, stands out due to simplicity and adaptability.
- **For Projects with Defined Release Cadence**: GitFlow offers a well-structured framework.
- **For Continuous Deployment/Integration Enthusiasts**: GitLab Flow's has great alignment with CI/CD.
- **For Advocates of Trunk-based Development**: GitLab Flow is the preferred choice.

## Conclusion

Selecting between GitLab Flow and GitFlow boils down to project specifics and team dynamics. Factors such as the team's Git expertise, project characteristics, release rhythms, and CI/CD commitments play pivotal roles. It's imperative for teams to assess the needs of their project and then chart their path. Both methodologies offer distinct strengths, making the choice more about alignment than superiority.

My recommendation for my team was to use Gitlab Flow since we already use Gitlab for our project. I wanted to take advantage of its built-in features like releases and environments to streamline our workflow and provide increased visibility into what and where code is deployed. While our project uses a defined(ish) release cadence, the overall goal is to get as close as possible to continuous delivery, and Gitlab Flow is flexible enough to adjust it to our release cadence.
