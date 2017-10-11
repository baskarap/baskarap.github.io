---
layout: post
title: After Sometime with Gitlab
---

Our devops just recently introduced Gitlab to replace our well-built CI/CD Jenkins (combined with Bitbucket as the repository). As a software engineer who only knows about coding (I was a mobile developer last year), I had no idea why were we investing our time to do a migration. Apart from that, Jenkins and Bitbucket combination seemed perfectly fine for me. But since we have a group of brilliant people in our devops team, they should have some reasons.

![Gitlab Icon]({{ site.url }}/images/gitlab-logo.png)

## A Single Located CI/CD Script

Believe it or not, we had 175 projects maintained by Jenkins at last time we were using it. I might be wrong, because some projects are probably inactive or deprecated because it has been rewritten using other language or splitted into some micro-services. But, I believe that we at least had around 100 services to be maintained. How did we use to maintain our CI/CD then?

Even though Jenkins has a nice UI that allows you to build pipelines by clicking this and clicking that, we did not use this way. We had one repository on bitbucket, which was called Jenkins-Jobs. There, we had all pipeline scripts written in Groovy language to build pipelines of a project. The reason why was mainly because if we were using Jenkins’ dashboard to configure pipelines, it would be difficult to track who made the changes and what changes being made in case something bad happened. Apart from that, we might have some customised CI jobs that only happened if we created by a script.

Everything seemed to be fine earlier until we started introducing many micro-services and products started growing. There were so many requests to our devops team to customise or build a new CI/CD pipeline whenever new service was needed. That was a hell of burden for them. Some people like me decided to just raise a PR to Jenkins-Jobs so devops would just need to review and merge it. But that was not enough because with our company’s speed of growth & change, devops would face tens of requests per day. Don’t forget that they also had to deal with production issues, new environment box requests, etc.

## A Testing Blocker

We used to have only master branch to be deployed. Whenever you want to have a specific pipeline for your branch, you will need to modify the Jenkins-Jobs project. It would obviously take time to be merged. And also it wouldn’t be efficient because you will always keep your branch alive until the feature gets released. So, why was this considered as a testing blocker?

Imagine you have a feature developed on your branch, and it is a huge one. Now you need to demo it to your Managers or QAs to test in staging environment without having it on master branch. We couldn’t achieve this using our approach with Jenkins. QAs would be only able to test once it was merged to master, which would be deployed to staging.

## Repository Access Problem

Since we had only one Jenkins-Jobs repository for all services/projects, whoever had write access into it, would be able to screw up every single project’s pipelines within my company. Obviously it wouldn’t happen unless you are an idiot or want to be fired. But then, risk was there.

## Gitlab (with it's CI) Came As The Saviour

What I love most from Gitlab is it’s CI, which lays in **gitlab-ci.yml** file. Every repository with this file will magically has a pipeline, depends on how you configure it. It decouples the dependency on Jenkins-Jobs project that we used to have. One stop solution (repository + CI/CD) completely blows Jenkins + repo service away. This also answers whenever you want to have a branch specific deployment, you can just add the script on this file and you will get your pipeline ready. Once you are going to get your branch merged, you can just checkout the file.

Apart from that, Gitlab has successfully solved repository problem that we had. Now, for example we have Group-A, Group-B, and Group-C groups (which contain some repositories). If i only have write access to Group-A, than I can only screw Group-A’s projects pipelines, not others. Risks are minimised here.

Gitlab also has a nice feature that prevents members to make changes to a particular branch (for example: only specific developers are able to push to master). It makes your life easier when you are working with a huge team.

Finally, all of these things are written from a software engineer who doesn’t really know about infrastructure and mystical things behind a CI/CD. There must be many other reasons why we decided to move from Jenkins + Bitbucket. But from my point of view, these points are major reasons why we are using Gitlab now. To see more about how Gitlab CI works, their documentation [here](https://docs.gitlab.com/ce/ci/quick_start/) is pretty sick.