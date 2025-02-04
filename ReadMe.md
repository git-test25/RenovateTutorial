# Executable Renovate-tutorial

## Introduction
Renovate is a dependency manager and its main use is to monitor all dependencies in a project and automatically update them according to your chosen preferences. For instance, Renovate bot will automatically create pull requests whenever dependencies need updating. Renovate supports a wealth of languages and is highly customizable. There are multiple options on how to set up and use Renovate. It is easily available if you are hosted at Github or Azure DevOps. For platforms such as Bitbucket Cloud, Bitbucket Server, Gitea and GitLab, Renovate can be used by self-hosting it. [Dependabot](https://dependabot.com/) is another dependency manager that is similar to Renovate. Dependabot is built to be simple to use and the trade off is therefore, it does not provide the same amount of configuration as Renovate does. Another difference is that Dependabot does not include auto-merging which is a very useful feature. More about auto-merging will be discussed further into the tutorial.

This tutorial will provide a brief introduction to the tool and how to set up Renovate bot for an example node application hosted on Github.

## Table of Contents
- Preparation
- Fork Example Application
- Install Renovate
- Configure Renovate

## Preparation
To complete this tutorial you will need a Github account and a web-browser.

## Fork Example Application

1. The example project we are going to use is a simple web server that listens to clients that connect to <http://localhost:3000>. When a client connects to the server, then the server sends the message "Hello World!" as a response. The example project can be found here <https://github.com/Sebberh/GenericNode>. For more details about the example project, check its README. Moving on to the first step of this tutorial, navigate to <https://github.com/Sebberh/GenericNode>

2. Fork the repository and enable issues on the fork.
![](images/2.png)

## Install Renovate

3. Navigate to <https://github.com/apps/renovate> and click the Install button
![](images/3.png).

4. Set the repository to either "All repositories" or just select the fork.
![](images/4.png)

5. Click Install.

6. Click Activate now and sign authorize with your Github account.
![](images/6a.png)
   The optional part of this step is to complete the WhiteSource Renovate Registration and get access to the Renovate Dashboard. The Renovate Dashboard includes all jobs that are done by Renovatebot for each repository. Click on a job to view its details and corresponding logs.

7. Go to your Fork

8. Go to Pull requests and open the pull request named Configure renovate
![](images/8.png)<br/>
It should look something like this:
![](images/8b.png)

9. Read through the configuration summary in the pull request. This pull request includes the configuration file for Renovate named `renovate.json`. The default base configuration for all languages looks like this

```
{
  "extends": [
    "config:base"
  ]
}
```

were `config:base` is the is a configuration preset of the following presets.

```
{
  "extends": [
    ":separateMajorReleases",
    ":combinePatchMinorReleases",
    ":ignoreUnstable",
    ":prImmediately",
    ":semanticPrefixFixDepsChoreOthers",
    ":updateNotScheduled",
    ":automergeDisabled",
    ":ignoreModulesAndTests",
    ":autodetectPinVersions",
    ":prHourlyLimit2",
    ":prConcurrentLimit20",
    "group:monorepos",
    "group:recommended",
    "helpers:disableTypesNodeMajor",
    "workarounds:all"
  ]
}
```

Here is a short description for some of the configurations which the base configuration includes:
  - `:prImmediately`: a pull request is created immediately after a branch is created
  - `:ignoreUnstable`: only allow upgrade to unstable versions if the existing version is unstable
  - `:ignoreModulesAndTests`: ignore `node_modules`, `bower_components`, `vendor` and various test/tests directories
  - `:autodetectPinVersions`: autodetect whether to pin dependencies or maintain ranges
  - `:automergeDisabled`: auto-merging feature disabled, only humans are allowed to merge pull requests
More details about the default configuration presets are provided in the [Renovate documentation](https://docs.renovatebot.com/presets-default/).

10. Merge the pull request to enable Renovate on your Fork. Wait a minute and a pull request will be created for updating the `node` dependency, merge the pull request to update the dependency.
![](/images/10.png)

## Configure Renovate and Enable Auto-merging

11. Navigate to the project-files and open renovate.json.

12. Overwrite the content with the following:

```
{
  "extends": [
    ":separateMajorReleases",
    ":combinePatchMinorReleases",
    ":ignoreUnstable",
    ":prImmediately",
    ":semanticPrefixFixDepsChoreOthers",
    ":updateNotScheduled",
    ":ignoreModulesAndTests",
    ":autodetectPinVersions",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone",
    "group:monorepos",
    "group:recommended",
    "helpers:disableTypesNodeMajor",
    "workarounds:all",

    ":pinAllExceptPeerDependencies"
  ]
}
```
This configuration is equivalent to the defaults (see description in the pull named Configure renovate) with four exceptions:
```
":prHourlyLimitNone"
```
  - No limit on how many pull requests are created per per hour. The default is 2, which can cause large delays when confirming a new config if the limit has been hit.
```
":prConcurrentLimitNone"
```
  - No limit on how many pull requests are created concurrently. The default is 20, which would probably not cause any problems for this tutorial but might cause delays if you experiment on your own.

  - Removing *":automergeDisabled"* allows for enabling automerge in the repo. This is necessary since we will configure auto-merge next.
```
":pinAllExceptPeerDependencies"
```
  - Pins all dependency versions except peer dependencies. This is recommended for Node and considered good practice

Pull requests will be made before any changes are made to the codebase.

13. Wait for a couple of minutes and the check your pull requests for a request named "Pin dependencies"

14. Open the pull request, it should look something like this:
![](images/14.png)

15. Merge the pull request and check that all versions have been pinned.
Example:
```
Unpinned                Pinned
"express": "^4.17.0"    "express": "4.17.0"  
```
Pinning, as opposed to using ranges, means that npm will use exactly ther version of the library that is specified. Ranges are more flexible and can (but does not necessarily) use newer versions. Exactly how they work depends on what prefix is used, ^ will allow minor version-upgrade by npm.
An advantage to pinning is that you can run tests before allowing even minor updates in production and you get more control of the environment.

16. Next, we'll break the config on purpose while setting up auto merge for minor updates. We make the configuration invalid by adding this

```
"packageRules": [
  {
    "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
    "requiredStatusChecks": null,
    "automerge": true
  }
]
```

to `renovate.json` within `extends`, the result should look like this:

```
{
  "extends": [
    ":separateMajorReleases",
    ":combinePatchMinorReleases",
    ":ignoreUnstable",
    ":prImmediately",
    ":semanticPrefixFixDepsChoreOthers",
    ":updateNotScheduled",
    ":ignoreModulesAndTests",
    ":autodetectPinVersions",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone",
    "group:monorepos",
    "group:recommended",
    "helpers:disableTypesNodeMajor",
    "workarounds:all",

    ":pinAllExceptPeerDependencies",

    "packageRules": [
      {
        "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
        "requiredStatusChecks": null,
        "automerge": true
      }
    ]
  ]


}
```
This configuration is invalid because the `packageRules` is not a valid object within `extends`.

17. Check your Issues and see that Renovate have created an issue stating that the config is broken, including an error-message. It should look something like this:
![](images/19.png)


18. To enable auto-merging of pull requests created by renovate, move the `packageRules` object out of `èxtends` and as a separate object in the configuration.

```
"packageRules": [
  {
    "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
    "requiredStatusChecks": null,
    "automerge": true
  }
]
```

The added package rule enables auto-merging for the repository. `matchUpdateTypes` defines which type of dependency updates auto-merging should be applied on, in this configuration auto-merging is performed when detecting minor dependency updates, updates for pinned dependencies, patches and updates for dependencies with no change/tag (digest). Note that `"requiredStatusChecks": null` disables the requirement of a successful run of the CI pipeline. This is disabled for the purpose of demonstration in this tutorial and the fact that the example project does not have a CI pipeline set up. In practice, it would be very reasonable to require a successful run of the CI pipeline before auto-merging.

The `renovate.json` file should look like this:

```
{
  "extends": [
    ":separateMajorReleases",
    ":combinePatchMinorReleases",
    ":ignoreUnstable",
    ":prImmediately",
    ":semanticPrefixFixDepsChoreOthers",
    ":updateNotScheduled",
    ":ignoreModulesAndTests",
    ":autodetectPinVersions",
    ":prHourlyLimitNone",
    ":prConcurrentLimitNone",
    "group:monorepos",
    "group:recommended",
    "helpers:disableTypesNodeMajor",
    "workarounds:all",
    ":pinAllExceptPeerDependencies"
  ],
  "packageRules": [
    {
      "matchUpdateTypes": ["minor", "patch", "pin", "digest"],
      "requiredStatusChecks": null,
      "automerge": true
    }
  ]
}
```

19. Check the issue again and see that the bot has closed it automatically.

20. Open package.json and change the version of express to 4.17.0. The file should look something like this:

```
{
  "dependencies": {
    "express": "4.17.0",
    "node": "16.0.0",
    "react": "17.0.2"
  },
  "name": "nodeexample",
  "version": "1.0.0",
  "description": "Test for tut",
  "main": "app.js",
  "devDependencies": {},
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Sebastian Fagerlind",
  "license": "ISC"
}
```
21. Wait a couple of minutes and check your closed pull requests and you will see that a pull request has been both created and merged automatically.

![](images/22.png)

While enabling auto-merge has some risks, problematic updates can get merged, but it can also help reduce the amount of noise and make managing major updates easier.
You have to make a decision based on your circumstances, are the risks of a bad minor update worth reducing the amount pull requests?

22. Explore the depths of configuration available at <https://docs.renovatebot.com/configuration-options/> at your own leisure.

## Summary

This tutorial go through:

* Activating Renovate on you repo and activate the renovate dashboard
* Pin and update your dependencies  
* Change the default configuration to allow automatic merges of minor version
* How to notice and fix an error in the configuration


## Take home message

Don't be afraid to experiment with configuration, but be prepared to exercise patience.

Always test your configuration and confirm that it does what you mean for it to do.

Never ***ever*** enable auto merging without testing before production.
