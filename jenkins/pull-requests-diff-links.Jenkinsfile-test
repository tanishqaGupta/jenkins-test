pipeline {
    agent any

    environment {
        //GIT_REPO_URL = 'https://github.com/tanishqaGupta/jenkins-test.git'
        //GIT_CREDENTIALS_ID = '5a83bda3-56b0-47df-82c0-95591a830f23'
        //TARGET_BRANCH = 'main'
        BITBUCKET_COMPARE_URL = 'https://bitbucket.costco.co.jp/projects/COSHYB/repos/costco-sip/compare/diff?targetBranch=refs%2Ftags%2F' 
        TARGET_KEY = 'charts.sip.version'
        FILE = "versions.yaml"
        LOCAL_BIN = "${WORKSPACE}/bin"
    }

    stages {
        /*stage('Clone Repository') {
            steps {
                git url: GIT_REPO_URL, credentialsId: GIT_CREDENTIALS_ID, branch: TARGET_BRANCH
            }
        }*/

        stage('Prepare Environment by installing yq') {
            steps {
                sh '''
                    mkdir -p $LOCAL_BIN

                    if ! command -v wget &> /dev/null; then
                        curl -L -o $LOCAL_BIN/wget https://ftp.gnu.org/gnu/wget/wget-1.21.1.tar.gz
                        tar -xzf $LOCAL_BIN/wget -C $LOCAL_BIN
                        chmod +x $LOCAL_BIN/wget
                    fi

                    curl -L https://github.com/mikefarah/yq/releases/download/v4.30.6/yq_linux_amd64 -o $LOCAL_BIN/yq
                    chmod +x $LOCAL_BIN/yq
                '''
            }
        }

        stage('Get Versions') {
            steps {
                script {
                    def currentVersion = sh (
                        script: "$LOCAL_BIN/yq e '.${TARGET_KEY}' ${FILE}",
                        returnStdout: true
                    ).trim()
                    echo "Current Version: ${currentVersion}"

                    def lineNumber = sh (
                        script: "$LOCAL_BIN/yq e '.${TARGET_KEY} | line' ${FILE}",
                        returnStdout: true
                    ).trim()
                    
                    def previousVersion = "N/A"

                    if (lineNumber) {
                        def lastCommitId = sh (
                            script: "git blame -L ${lineNumber},${lineNumber} ${FILE} | awk '{print \$1}'",
                            returnStdout: true
                        ).trim()

                        previousVersion = sh (
                            script: "git show ${lastCommitId}^:${FILE} | $LOCAL_BIN/yq e '.${TARGET_KEY}' -",
                            returnStdout: true
                        ).trim()
                                        
                    if (currentVersion && previousVersion && currentVersion != previousVersion) {
                        currentBuild.displayName = "Version updated to ${currentVersion}"

                        def compareUrl = "${BITBUCKET_COMPARE_URL}${previousVersion}&sourceBranch=refs%2Ftags%2F${currentVersion}&targetRepoId=37"
                        echo "Comparison URL: ${compareUrl}"
                        
                    } else {
                        echo "Key '${TARGET_KEY}' not found in file '${FILE}'."
                    }

                    echo "Current Version: ${currentVersion}"
                    echo "Previous Version: ${previousVersion}"
                    }
                }
            }
        }
    }
}
