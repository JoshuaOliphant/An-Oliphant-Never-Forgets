---
layout: post
title: How to get the status of a Gitlab child pipeline
date: 2023-08-23
catagories: Gitlab
---
I recently needed to get the status of a child pipeline in Gitlab so that I could send the result to a Slack channel. After looking at the documentation for pipelines, it clearly states that the endpoint that lists the pipelines for a project does not include child pipelines. It even has a source query where you could say:

/pipelines/:pipeline_id?source=trigger
Or
/pipelines/:pipeline_id?source=parent_pipeline

And it still will not give you the child pipelines. The documentation does say that you can get a child pipeline individually, but without being able to list them how would I know the id of the pipeline to query in the first place?

I did some digging and came across a number of unsatisfying solutions. Eventually, I came across this 3 year old Gitlab issue where the folks at Gitlab added a endpoint that looks like:

/pipelines/:pipeline/bridges

This returns a list of “bridge” jobs, which are the jobs that trigger downstream pipelines. Each job helpfully has the name of the job, the job status, and downstream pipeline objects with it’s id. Using this, I set up a job that runs immediately after this bridge job and reports its status to Slack.
