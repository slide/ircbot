# JIRA/GitHub management IRCBot

[![Build Status](https://ci.jenkins.io/job/Infra/job/ircbot/job/master/badge/icon)](https://ci.jenkins.io/job/Infra/job/ircbot/job/master/)

This IRC bot sits on `#jenkins` as `jenkins-admin` and allow users to create/fork repositories on GitHub, etc. More info: [Jenkins IRC Bot Page](https://jenkins.io/projects/infrastructure/ircbot/)

## Deployment
This repo is containerized, then deployed to our infrastructure via Helm. 
You should have a Write permission to https://github.com/jenkins-infra/ircbot and https://github.com/jenkins-infra/jenkins-infra to deploy the new version of the Bot.

Actions:

1. Commit/merge changes into the master branch
2. Wait till the preparation of Docker package on Jenkins INFRA 
 * The container is built on the Trusted Infrastructure, the best way to determine when the new tag is ready is to look at dockerhub tags
 * https://hub.docker.com/r/jenkinsciinfra/ircbot/tags
 * There should be a new tag within 30 minutes of the merge/commit to the master branch
4. Modify the version in the Helm values.yaml file
 * Create a PR for the https://github.com/jenkins-infra/charts/blob/master/charts/chatbot/values.yaml file
 * Change the `tag` value to the tag from dockerhub
 * Wait till the merge of the pull request.
5. Wait till the deployment. 
   it is possible to call the `jenkins-admin: version` command to check the current version

## License
[MIT License](https://opensource.org/licenses/mit-license.php)

## Developer guide

This section contains some info for developers.

### Reusing IRCBot in non-Jenkins project

The bot is designed to be used in Jenkins, but it can be adjusted in other projects, 
which use the similar infrastructure (GitHub, IRC, JIRA). 
Adjustements can be made via System properties.
These properties are located and documented in the 
<code>org.jenkinsci.backend.ircbot.IrcBotConfig</code> class.

Several examples are provided below.

### Building the bot

0. Use Maven to build the project and to run the unit tests.
0. Then use Dockerfile to create a Docker image

For detailed examples see [Jenkinsfile](Jenkinsfile) located in this repository.

### Testing the bot locally

Preconditions:

0. You have a JIRA **Test** Project, where you have admin permissions.
1. You have a GitHub Organization with ```Administer``` permissions

Setting up the environment:

0. Setup Github credentials in the ```~/.github``` file
 * Format: Java properties
 * Entries to set: ```login``` and ```password```
 * It's also possible ```oauth``` and ```endpoint``` properties 
 (see [github-api](https://github.com/kohsuke/github-api))
1. Setup JIRA credentials in the ```~/.jenkins-ci.org``` file
 * Format: Java properties
 * Entries to set: ```userName``` and ```password```

Running the bot for testing:

```
java -Dircbot.name=test-ircbot \ 
-Dircbot.channels="#jenkins-ircbot-test" \ 
-Dircbot.testSuperUser="${YOUR_IRC_NAME}" \ 
-Dircbot.github.organization="jenkinsci-infra-ircbot-test" \
-Dircbot.jira.url=${JIRA_URL} \
-Dircbot.jira.defaultProject=TEST \
-jar target/ircbot-2.0-SNAPSHOT-bin/ircbot-2.0-SNAPSHOT.jar 
```
   
After executing this command the bot should connect to your IRC chat.
