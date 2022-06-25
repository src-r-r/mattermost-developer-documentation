---
title: "Server Workflow"
heading: "Mattermost Server Workflow"
description: "See what a general workflow for a Mattermost developer working on the mattermost-server repository looks like."
date: 2017-08-20T11:35:32-04:00
weight: 3
---

If you haven't [set up your developer environment](https://developers.mattermost.com/contribute/server/developer-setup/), please do so before continuing with this section.

Join the [Developers community channel](https://community.mattermost.com/core/channels/developers) to ask questions from community members and the Mattermost core team.

### Workflow

Here's a general workflow for a Mattermost developer working on the [mattermost-server](https://github.com/mattermost/mattermost-server) repository:

1. Review the repository structure to familiarize yourself with the project
    * [./api4/](https://github.com/mattermost/mattermost-server/tree/master/api4) holds all API and application related code.
    * [./model/](https://github.com/mattermost/mattermost-server/tree/master/model) holds all data model definitions and the Go driver.
    * [./store/](https://github.com/mattermost/mattermost-server/tree/master/store) holds all database querying code.
    * [./utils/](https://github.com/mattermost/mattermost-server/tree/master/utils) holds all utilities, such as the mail utility.
    * [./i18n/](https://github.com/mattermost/mattermost-server/tree/master/i18n) holds all localization files for the server.
2. On your fork, create a feature branch for your changes. Name it `MM-$NUMBER_$DESCRIPTION` where `$NUMBER` is the [Jira](https://mattermost.atlassian.net) ticket number you are working on and `$DESCRIPTION` is a short description of your changes. Example branch names are `MM-18150_plugin-panic-log` and `MM-22037_uppercase-email`.
3. Make the code changes required to complete your ticket, ensuring that unit tests are written or modified where appropriate. Please use [testify](https://github.com/stretchr/testify) for new tests.
4. If you made changes to the store, run `make store-mocks` and `make store-layers` to update test mocks and timing layer.
5. To test your changes, run `make run-server` from the root directory of the server respository. This will start up the server at `http://localhost:8065`. To get changes to the server it must be restarted with `make restart-server`. If you want to test with the web app, you may also run `make run` which will start the server and a watcher for changes to the web app.
6. Once everything works to meet the ticket requirements, stop Mattermost by running `make stop` in the server repository, then run `make check-style` to check your syntax.
7. Run the tests using one or more of the following options:
     * Run `make test` to run all the tests in the project. This may take a long time and provides very little feedback while it's running.
     * Run individual tests by name executing `go test -run "TestName" ./<directory>`.
     * Run all the tests in a package where changes were made executing `go test app`.
     * Create a draft PR with your changes and let our CI servers run the tests for you.
8. If you added or changed any localization strings you will need to run `make i18n-extract` to generate the new/updated strings.
9. Commit your changes, push your branch, and [create a pull request](https://developers.mattermost.com/contribute/getting-started/contribution-checklist/).
10. Once a PR is submitted it's best practice to avoid rebasing on the base branch or force-pushing. Jesse, a developer at Mattermost, mentions this in his blog article [Submitting Great PRs](https://mattermost.com/blog/submitting-great-prs/). When the PR is merged, all the PR's commits are automatically squashed into one commit, so you don't need to worry about having multiple commits on the PR.
11. That's it! Rejoice that you've helped make Mattermost better.

### Useful Server Makefile Commands

Some useful `make` commands:

* `make run` runs the server, creates a symlink for your mattermost-webapp folder, and starts a watcher for the web app.
* `make stop` stops the server and the web app watcher.
* `make run-server` runs only the server and not the client.
* `make debug-server` will run the server in the `delve` debugger.
* `make stop-server` stops only the server.
* `make clean-docker` stops and removes your Docker images and is a good way to wipe your database.
* `make clean` cleans your local environment of temporary files.
* `make config-reset` resets the `config/config.json` file to the default.
* `make nuke` wipes your local environment back to a completely fresh start.
* `make package` creates packages for distributing your builds and puts them in the `./dist` directory. First you will need to run `make build` and `make build-client`.

If you would like to run the development environment without Docker you can set the `MM_NO_DOCKER` environment variable. If you do this, you will need to set up your own database and any of the other services needed to run Mattermost.

### Running Only Specific Server Unit Tests

Running every single unit test takes a lot of time while making changes, so you can run a subset of the serverside unit tests by using the following:

```
go test -v -run='<test name or regex>' ./<package containing test>
```

For example, if you want to run `TestUpdatePost` in `app/post_test.go`, you would execute the following:

```
go test -v -run='TestUpdatePost' ./app
```

### Useful Mattermost Commands

During development you may want to reset the database and generate random data for testing your changes. For this purpose, Mattermost has the following commands in the Mattermost CLI:

Install the server with `go install ./cmd/mattermost` in the server repository.

You can reset your database to the initial state using:

```
mattermost db reset
```

The following commands need to be run via our [mmctl](https://docs.mattermost.com/manage/mmctl-command-line-tool.html) tool.

You can generate random data to populate the Mattermost database using:

```
mmctl sampledata
```

Create an account using the following command:

```
mmctl user create --email user@example.com --username test1 --password mypassword
```

Optionally, you can assign that account System Admin rights with the following command:

```
mmctl user create --email user@example.com --username test1 --password mypassword --system_admin
```

### Customize your workflow

#### Makefile

You can customize variables of the Makefile by creating a `config.override.mk` file or setting environment variables. To get started, you can copy the `config.mk` file to `config.override.mk` and change the values in your newly copied file.

#### Docker-compose

If you create a `docker-compose.override.yaml` file at the root of the project, it will be automatically loaded by all the `Makefile` tasks using `docker-compose`, allowing you to define your own services or change the configuration of the ones mattermost provides.

### Testing with CircleCI

Testing locally with CircleCI can determine early if there are any issues that didn't come up in testing.

These directions were written for Ubuntu 20.10.

First ensure all mattermost project (i.e. `mattermost-server`, `mattermost-webapp`) are found at `~/mattermost/`. This is so that circleci can mount them.

Install circleci. Execute the following command below, as this specific version is important (see https://github.com/CircleCI-Public/circleci-cli/issues/676).

```bash
curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | VERSION=0.1.16947 sudo -E bash
```

Create a new file `vim ~/.circleci/build_agent_settings.json` and add the following text (this is important for the picard docker image):

```
{"LatestSha256":"sha256:008ba7f4223f1e26c11df9575283491b620074fa96da6961e0dcde47fb757014"}
```


Ensure at least `docker-ce` and `docker-ce-cli` are installed by following the directions here: https://docs.docker.com/engine/install/

Within the `mattermost-server` directory check that circleci is working properly:

```bash
circleci orb validate .circleci/config.yml
```
Within the `mattermost-server` directory execute the following to run the tests:

```bash
circleci local execute --config=.circleci/config.yml
```

You should start to see the following output:

```
Fetching latest build environment...
Docker image digest: sha256:78f4c22eb8faa4c0bcf095fc1eecf4d53ef5fed32700f37900bbcb59ffa617ac
====>> Spin up environment
Build-agent version  ()
System information:
 Server Version: 20.10.14
 Storage Driver: overlay2
  Backing Filesystem: extfs
 Cgroup Driver: cgroupfs
 Cgroup Version: 2
 Kernel Version: 5.10.104-linuxkit
 Operating System: Docker Desktop
 OSType: linux
 Architecture: x86_64

Starting container mattermost/mattermost-build-server:20220415_golang-1.18.1
Warning: No authentication provided, using CircleCI credentials for pulls from Docker Hub.
  image cache not found on this host, downloading mattermost/mattermost-build-server:20220415_golang-1.18.1
```

If you see errors about not being able to mount the config file, open docker-desktop settings. Go to **Resources** > **File Sharing**, and add `/tmp` as a mounted resource. Finally, click **Apply & Restart** to save changes.

![image](https://user-images.githubusercontent.com/1633901/175781542-123d0cbb-0f0f-4e1f-91fa-32951fc6069b.png)

### Testing Email Notifications

When Docker starts, the SMTP server is available on port 2500. A username and password are not required.

You can access Inbucket webmail on port 9000.

For additional information on configuring an SMTP email server, including troubleshooting steps, see [https://docs.mattermost.com/install/smtp-email-setup.html](https://docs.mattermost.com/install/smtp-email-setup.html).

### Testing with GitLab Omnibus

To test a locally compiled version of Mattermost with GitLab Omnibus, replace the following GitLab files:

* The compiled `mattermost` binary in `/opt/gitlab/embedded/bin/mattermost`.
* The assets (templates, i18n, fonts, webapp) in `/opt/gitlab/embedded/service/mattermost`.
