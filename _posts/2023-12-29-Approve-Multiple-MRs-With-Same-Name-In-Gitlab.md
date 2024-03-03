---
layout: post
title: Script to Approve Multiple MRs with the Same Branch Name Across Multiple Gitlab Projects
date: 2023-12-29
categories: [Python, Click, Trogon, ChatGPT, Automation]
---
[Gist here](https://gist.github.com/JoshuaOliphant/72013ca894377883535f35967ab25028)

```python
import click
import gitlab

@click.command()
@click.option('--token', prompt=True, help='Your GitLab personal access token.')
@click.option('--gitlab_url', default='https://gitlab.com', help='Your GitLab instance URL.')
@click.option('--mr_name', prompt=True, help='The name of the merge requests to approve.')
def approve_merge_requests(token, gitlab_url, mr_name):
    # Initialize GitLab client
    gl = gitlab.Gitlab(url=gitlab_url, private_token=token)

    # List all projects
    projects = gl.projects.list(all=True)
    
    for project in projects:
        # List merge requests for each project
        merge_requests = project.mergerequests.list(state='opened')
        click.echo(f"Merge requests for project {project.name}:")
        click.echo(f"--------------------------------")
        for mr in merge_requests:
            click.echo(f"MR: {mr}")
            click.echo(f"Title: {mr.source_branch}")
            # Check if the merge request title matches the specified name
            if mr.source_branch == mr_name and project.name != "lrs-service-amdocs":
                # Approve the merge request
                mr.approve()
                click.echo(f"Approved MR: {mr.title} in project {project.name}")

if __name__ == '__main__':
    approve_merge_requests()
```
