pipelineJob("testing/pr-diff-links-test") {
    agent {
        kubernetes {
            defaultContainer com.costco.Agents.DEPLOYMENT_DEFAULT_AGENT_CONTAINER
            yaml com.costco.Agents.DEPLOYMENT_DEFAULT_AGENT
        }
    }
    properties {
        configure stashBuildTriggerConfig(
                "COSHYB",
                "environments/dev-.*",
                "app-configuration"
        )
    }

    definition {
        cpsScm {
            scm {
                git {
                    remote {
                        url "https://bitbucket.costco.co.jp/scm/con/app-configuration.git"
                        credentials "pipelines-ci-pwd"
                        refspec "+refs/pull-requests/*:refs/remotes/origin/pull-requests/*"
                    }
                    branches 'environments/dev-SIP-40286-pr-diff-link-testing'
                }
            }
            scriptPath "jenkins/pull-requests-diff-links.Jenkinsfile"
          	lightweight(true)
        }
    }
}

Closure stashBuildTriggerConfig(String repoProject, String branches, String repoName) {
    return {
        it / "properties" / "org.jenkinsci.plugins.workflow.job.properties.PipelineTriggersJobProperty" {
            triggers {
                "stashpullrequestbuilder.stashpullrequestbuilder.StashBuildTrigger"("plugin": "stash-pullrequest-builder@1.17") {
                    'spec'('H/2 * * * *')
                    'cron'('H/2 * * * *')
                    'stashHost'('https://bitbucket.costco.co.jp/')
                    'credentialsId'('pipelines-ci-pwd')
                    'projectCode'(repoProject)
                    'repositoryName'(repoName)
                    'ignoreSsl'('false')
                    'targetBranchesToBuild'(branches)
                    'checkDestinationCommit'('true')
                    'checkNotConflicted'('true')
                    'checkMergeable'('false')
                    'checkProbeMergeStatus'('true')
                    'mergeOnSuccess'('false')
                    'deletePreviousBuildFinishComments'('false')
                    'cancelOutdatedJobsEnabled'('false')
                    'ciSkipPhrases'('NO TEST')
                    'onlyBuildOnComment'('false')
                    'ciBuildPhrases'('test this please')
                }
            }
        }
    }
}
