pipeline {
    agent {
        kubernetes {
            defaultContainer com.costco.Agents.DEPLOYMENT_DEFAULT_AGENT_CONTAINER
            yaml com.costco.Agents.DEPLOYMENT_DEFAULT_AGENT
        }
    }

    environment {
        GIT_REPO_URL = 'https://bitbucket.costco.co.jp/scm/con/app-configuration.git'
        GIT_CREDENTIALS_ID = 'pipelines-ci-pwd'
        FILE_TO_MONITOR = 'versions.yaml'
        BITBUCKET_COMPARE_URL = 'https://bitbucket.costco.co.jp/projects/COSHYB/repos/costco-sip/compare/diff?targetBranch=refs%2Ftags%2F' 
        TARGET_BRANCH = 'environments/dev-SIP-40286-pr-diff-link-testing'
        PREVIOUS_VERSION_FILE = 'previous_version.txt'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: GIT_REPO_URL, credentialsId: GIT_CREDENTIALS_ID, branch: TARGET_BRANCH
            }
        }

        stage('Check for Version Change') {
            steps {
                script {
                    // Read the current version from version.yaml in the latest commit
                    def currentVersionFile = readYaml file: FILE_TO_MONITOR
                    def currentVersion = currentVersionFile.version
                    echo "Current version: ${currentVersion}"

                    // Get the last two commits that modified version.yaml
                    def versions = sh (
                        script: "git log -2 --pretty=format:%H -- \${FILE_TO_MONITOR} | xargs -I {} sh -c \"git show {}:\${FILE_TO_MONITOR} | grep 'version:' | sed 's/version: //g'\"",
                        returnStdout: true
                    ).trim().split('\n')

                    // Define the previous and second-to-last versions
                    def previousVersion = versions.size() > 0 ? versions[0].trim() : null
                    def lastVersion = versions.size() > 1 ? versions[1].trim() : null

                    echo "Previous version: ${previousVersion}"
                    echo "Last version: ${lastVersion}"

                    // Construct the URL for comparing the last and second-to-last versions
                    if (previousVersion && lastVersion && previousVersion != lastVersion) {
                        echo "Version has changed. Triggering job actions..."
                        currentBuild.displayName = "Version updated to ${previousVersion}"

                        def compareUrl = "${BITBUCKET_COMPARE_URL}${lastVersion}&sourceBranch=refs%2Ftags%2F${previousVersion}&targetRepoId=37"
                        echo "Comparison URL: ${compareUrl}"

                    } else {
                        echo "No version change detected between the last two commits."
                    }
                }
            }
        }
    }
}
