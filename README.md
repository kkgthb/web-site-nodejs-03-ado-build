# Your first Azure DevOps build pipeline

(Accompanies associated blog post on Katie Kodes "[Making Azure DevOps Pipelines build a Hello World webapp from Git-tracked source code changes](https://katiekodes.com/ado-build-hello-world-from-git/)".)

---

1. Please work your way through my entire exercise "[Source code that builds locally into a Node.js Hello World webapp](https://katiekodes.com/node-hello-world/)" before beginning this exercise so that you're familiar with "building" and "running" a webserver _(although in this case, we'll only do the "building" part)_ from source code.
1. Have an account in Azure DevOps _("ADO")_.  You can get one:
    * From Microsoft for free but [with a credit card on file](https://azure.microsoft.com/en-us/free/).
    * From Microsoft for free [without a credit card as a student](https://azure.microsoft.com/en-us/free/students).
    * From Microsoft [without a credit card through a Visual Studio subscription if you have one at work](https://azure.microsoft.com/en-us/pricing/member-offers/credit-for-visual-studio-subscribers/).
    * For 4 hours at a time through an [A Cloud Guru subscription](https://acloudguru.com/platform/cloud-sandbox-playgrounds) _(personally, I'd rather get 1 yearly charge from ACG than give Microsoft my credit card and expose myself to getting hacked and running up an "infinity" bill)_.

No need to install anything onto your computer -- we'll do everything in this article through a web browser.

---

## Importing my sample codebase into ADO Repos

1. [Log into Azure](https://portal.azure.com/).
1. Click into "[Azure DevOps Organizations](https://portal.azure.com/#blade/AzureTfsExtension/OrganizationsTemplateBlade)" from the Azure home screen.
1. Click "[My Azure DevOps Organizations](https://aex.dev.azure.com/)."
1. Click into an ADO Organization you created earlier, or click the "Create new organization" button if you don't have any yet.
    * You should be taken to a website like `https://dev.azure.com/YOUR-ADO-ORG-NAME/`.
1. Click into an ADO Project you created earlier, or click the "New project" button if you don't have any yet.
    * You should be taken to a website like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/`.
1. In the left-hand navigation, click on "**Repos**."
    * Alternatively, visit a URL like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_code/`, making appropriate org & project name substitutions.
1. Up at the top, where the navigation breadcrumb says something like "`YOUR-ADO-ORG-NAME` / `YOUR-ADO-PROJECT-NAME` / Repos / Files / `SOME-REPOSITORY-NAME`, click `SOME-REPOSITORY-NAME` to expose a dropdown picklist letting you choose a different repository or create a new one.
1. Click "**Import repository**."  In the flyout panel at right:
    * Set "**Repository type**" to "**Git**."
    * Set "**Clone URL**" to "**`https://github.com/kkgthb/web-site-nodejs-03-ado-build.git`**."
    * Set "**Name**" to whatever you'd like to name your new repository.
    * Click the "**Import** button in the bottom right corner of the flyout panel.
1. This would be a great time to [protect the `main` branch of your new repository from direct edits](https://katiekodes.com/ado-repo-branch-protection/){:target="_blank}, forcing you to [use new branches and pull requests](https://blog.mergify.com/github-branch-protection-what-it-is-and-why-it-matters/) instead.

---

## Telling Azure Pipelines to build when code changes

In [this series's kickoff article](https://katiekodes.com/node-hello-world/), we ran the operating system command "`npm run build`" on our own personal computer after installing Node.js and NPM onto it.

In this article, we'll borrow Microsoft's computers instead and do everything through our web browser.

### How the pipeline will behave

The YAML-formatted file "[`/cicd/my-azure-build-pipeline.yml`](https://github.com/kkgthb/web-site-nodejs-03-ado-build/blob/main/cicd/my-azure-build-pipeline.yml)" _(from the [sample codebase you just copied](https://github.com/kkgthb/web-site-nodejs-03-ado-build/))_ forces any ADO Pipeline running it to walk through the following steps sequentially:

1. Execute only if edits have occurred to the "`main`" branch of our ADO Repository.  Otherwise, skip the rest of the steps.
1. Borrow a short-lived Linux _(Ubuntu)_ computer _("[agent](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents?view=azure-devops)")_ from Microsoft.
1. Install Node.js version 18 _(and, implicitly, NPM)_ onto the agent.
1. Install the specific Node "modules" mentioned "`dependencies`" and `devDependencies`" in the file "[`/package.json`](https://github.com/kkgthb/web-site-nodejs-03-ado-build/blob/main/package.json)" onto the agent.  It does this by executing an "`npm install`" command on the agent's operating system.
1. Build the source code from "[`/src/web/`](https://github.com/kkgthb/web-site-nodejs-03-ado-build/blob/main/src/web/)" into a "`/dist/`" folder on the agent's operating system.  Like it did in the series kickoff on our own computer, `/dist/` will contain a "`server.js`" file and a "`node_modules`" subfolder.  That said, you won't be able to see the contents of `/dist/` yourself, since the "agent" _(the computer our pipeline borrows from Microsoft)_ will self-destruct as soon as the pipeline finishes running.
1. Copy the contents of "`/dist/`" to a slightly safer place _(a built-in ADO Pipelines location called "`$(Build.ArtifactStagingDirectory)`")_.
1. "Publish" the contents of "`$(Build.ArtifactStagingDirectory)`" into an even safer place:  a **long-lived** "ADO Pipelines **Artifact**" called "`MyBuiltWebsite`."
    * _(It won't be around forever, but by default if you didn't otherwise change your ADO Project settings, it'll probably stick around for about a month.)_

After these steps finish executing, it's safe for Azure DevOps to let the agent _(the Linux computer we borrowed from Microsoft)_ self-destruct.

### Forcing ADO to use our YAML pipeline code

1. In the left-hand navigation of Azure DevOps, click on "**Pipelines**."
1. Beneath that, click "**Pipelines**."
    * _(It takes you to a URL like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_build`.)_
1. Click "**New Pipeline**" _(upper right)_ if it's not your first, or "**Create Pipeline**" _(center)_ if it is your first pipeline.
1. Select the "**Azure Repos Git**" option.
1. Enter **the name of the repository you created** when you imported a copy of my codebase.
1. Select the "**Existing Azure Pipelines YAML file**" option.
1. Set the "**Path**" to "`/cicd/my-azure-build-pipeline.yml`" by choosing it from the dropdown picklist.
1. Click "**Continue**."
1. In the upper right corner above the YAML editor, drop down the picklist _**next to**_ "run" but be careful not to click "run" itself.  Instead, choose "**Save**" below it.
1. Please ignore the temptation to click the "Run pipeline" button just yet.  Don't click it.

---

## Running the pipeline by editing code

1. In the left-hand navigation of Azure DevOps, click on "**Repos**" and then "**Files**."  Click on the `README.md` file and click the "**Edit**" button in the upper-right corner above the file's contents.
1. Add some text anywhere in the readme, just so you'll have changed something.
1. Click the "**Commit**" button in the upper-right corner above the file editor panel.
1. In the flyout panel at right:
    * Change "**Branch name**" from "`main`" to "`small-branch`."
    * Check the box next to "**Create a pull request**."
    * Click the "**Commit**" button in the lower-right corner.
1. In the "**New pull request**" page, edit the **Title** and **Description** if you'd like and click the "**Create**" button at bottom-right.
    _(Note:  the "new pull request" page will live at some URL like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_git/YOUR-REPOSITORY-NAME/pullrequestcreate?sourceRef=small-branch&targetRef=main&sourceRepositoryId=SOME-BIG-HEXIDECIMAL-NUMBER&targetRepositoryId=SOME-BIG-HEXIDECIMAL-NUMBER`.)_
1. In the page titled after the "**Title**" you chose for the pull request, toward the upper-right, click the "**Approve**."
    _(Note:  the "pull request" will get its own URL like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_git/YOUR-REPOSITORY-NAME/pullrequest/SOME-INTEGER`.)_
1. Once you've done so, you should be able to click a "**Complete**" button next to "Approve."  Click it.
1. In the flyout panel at right, leave the settings as they are _(including deleting the "`small-branch`" branch after merging)_ and click the "**Complete merge**" button at the bottom-right.
1. You'll stay on the pull request page but see a green "Completed" tag in the upper left under its title.
1. Go back to look at your README _(`https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_git/YOUR-REPOSITORY-NAME?path=/README.md`)_ and note that your changes are in the "`main`" branch's copy of the README.

---

## Validating the pipeline ran correctly

1. In the left-hand navigation of Azure DevOps, click on "**Pipelines**."
1. Beneath that, click "**Pipelines**."
1. In the "**Recent**" tab titled "**Recently run pipelines**," you should see an entry, at the top of the list, from just a few minutes ago, named after your repository.  Click it.
1. In the "**Runs"** tab of the page named after your repository, click the top entry in the list.
    _(Note:  this particular pipeline's "runs" dedicated URL will be something like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_build?definitionId=SOME-INTEGER`.)_
1. In the "**Summary**" tab, you should be able to see that it ran against the "`main`" branch of `YOUR-REPOSITORY-NAME`, the time it started at, and how long it took to run.
    * There might be another list of runs between the "Recently run pipelines" and the "Summary" if your pipeline is still running.  Go ahead and click on the one that's running to get to it
    * _(Note:  this particular pipeline run's dedicated URL will be something like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_build/results?buildId=SOME-INTEGER&view=results`.)_
1. In the third column, titled "**Related**," click the link that says "**1 published**."
1. In the page called "**Artifacts**," expand click "`MyBuiltWebsite`" to expand it and see that it contains a "`server.js`" file and a "`node_modules`" subfolder.
    * CONGRATULATIONS, you convinced Azure DevOps Pipelines to build a runtime out of your latest source code.
    * PARTY!
    * _(Note:  this particular pipeline run's dedicated URL will be something like `https://dev.azure.com/YOUR-ADO-ORG-NAME/YOUR-ADO-PROJECT-NAME/_build/results?buildId=SOME-INTEGER&view=artifacts&pathAsName=false&type=publishedArtifacts`.)_
1. Go back to the pipeline run summary and scroll down to the box labeled "**Jobs**."  Click the entry in it titled "**Job**."
1. Click the various steps of the pipeline's execution history to see logs from each step.
    * A lot of the output probably looks a lot like the output you saw on your own computer when you worked through the exercise in [this series's kickoff article](https://katiekodes.com/node-hello-world/).  Computers borrowed from Microsoft probably aren't all too different from your own computer for simple jobs.

---

## Looking forward

Pat yourself on the back:  you have now built your first "CI/CD build pipeline" on Azure DevOps.

You might notice that we never did anything that actually _ran_ the server by executing a "`node ./dist/server.js`" command -- don't worry, we'll get there, but not until the very last article in this series.  Stay tuned!
