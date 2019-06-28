---
title: Trigger gitlab pipelines across projects
---

Did you want to trigger e2e tests for you project and didn’t find a straight way? Here is how.

![Pipelines]({{site.baseurl}}/public/img/gitlab/pipelines.jpeg)

You are probably right, there is not straight way in the gitlab community edition to acheive the wanted behavior. Most of the time, you don’t just want to trigger a pipeline another project, you also want to wait for the result of that pipeline, it means you want your pipeline to fail if the other one has failed.

## How could we do that?

The answer is: Gitlab APIs. Gitlab provides an API for pipelines that you can use it to trigger a pipeline in another project, if you have a trigger token created for that project. Also, you can get the status of a pipeline by it’s id, if you have `Personal Access Token`.

## How do i get a Trigger Token & an Personal Access Token?

Gitlab provides a good documentation about creating Triggers and their tokens, check it, and then comeback here to get the Personal Access Token .

## Getting Personal Access Token
In Gitlab, click on your profile icon in top right corner, then navigate to `settings` page.

On the leftside, you can see a sidebar full of settings menu, do you see `Access Tokens`? Click on that.

![]({{site.baseurl}}/public/img/gitlab/gitlab_access_tokens.png)

Now it’s time to create a new token, give it a name, and don’t forget to check the api box. click `Create personal access token`, gitlab will create new one that you can copy, **Be careful** gitlab will warn you that you will never see this token again, so you should keep it somewhere safe. Copy it and i would suggest to add it to your project CI/CD environment variables that we can use it later in CI.

    1. Go to your project settings, the one that you will trigger the pipeline from.

    2. Navigate to CI/CD settings, then expand the “Environment variables” section.

    3. By now you have two tokens, one for the trigger and one for the api, add both of them to the environment variables and save the.


![]({{site.baseurl}}/public/img/gitlab/gitlab_env.png)

The `PROJECT_ID`, it’s needed to point out which pipeline you want to trigger, and you can find the project id in the homepage of any project, like this:


![]({{site.baseurl}}/public/img/gitlab/gitlab_triggers.png)

## Time to trigger a pipeline
If you followed the gitlab documentation you will find out that you can use curl to do that, but then we said we also wanted to keep watching status for that pipeline to get the result. That is also doable in bash, but why the hassle.

I have wrote a small tool in Golang that could do both for us, not just that, I stuffed it in a very small docker image so that we could use it in gitlab pipelines.

take a look: https://gitlab.com/schehata/gitlab-pipeline-trigger

It’s very easy to use, we will just add it to your gitlab ci yaml file to trigger another pipeline, Remember when we added tokens and project id to our environment variables ? It’s time to use them.

So here is how would it look in gitlab-ci yaml file:

```yaml
e2e:
 stage: test
 image: registry.gitlab.com/schehata/gitlab-pipeline-trigger
 script:
   - /trigger/main -a $API_TOKEN -p $PROJECT_ID -t $TRIGGER_TOKEN
```

**Notice:** don’t forget to change $API_TOKEN, $PROJET_ID and TRIGGER_TOKEN to the names you specified in the project’s CI/CD settings.

As you can see, Iam setting the runner to use that image, and in the script i am using the `/trigger/main` golang tool to trigger the pipeline while passing the needed attributes (tokens and project id).

If you got the tokens and project id correctly and run the pipeline, you would see something like that:


![]({{site.baseurl}}/public/img/gitlab/gitlab_console.png)

Let me know how did it work out for you !

**That’s all folks !**
