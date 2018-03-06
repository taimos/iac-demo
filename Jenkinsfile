node {
    def version
    try {
        stage('Notify') {
            echo "Build & Test IaC Demo (${env.BUILD_URL})"
        }
        stage('Preparation') {
            checkout scm

            version = sh(script: 'git rev-list --all --count', returnStdout: true).trim()
            echo "Building version ${version}"
        }
        stage('Validate') {
            withAWS(region: 'eu-central-1', role: 'MrJenkins') {
                cfnValidate(file: 'cfn.yaml')
            }
        }
        if (env.BRANCH_NAME == 'master') {
            stage('Integration') {
                withAWS(region: 'eu-central-1', role: 'MrJenkins') {
                    def stack = cfnUpdate(stack: 'iac-demo-int', file: 'cfn.yaml', params: ['Environment': 'int', 'Version': version])
                    echo stack.toString()
                }
            }
            input
            stage('Production') {
                withAWS(region: 'eu-central-1', role: 'MrJenkins') {
                    def stack = cfnUpdate(stack: 'iac-demo-prod', file: 'cfn.yaml', params: ['Environment': 'prod', 'Version': version])
                    echo stack.toString()
                }
            }
        } else {
            stage('Testsetup') {
                withAWS(region: 'eu-central-1', role: 'MrJenkins') {
                    def stackName = "iac-demo-${env.BRANCH_NAME}"

                    def stack = cfnUpdate(stack: stackName, file: 'cfn.yaml', params: ['Environment': env.BRANCH_NAME, 'Version': version])
                    echo stack.toString()

                    try {
                        def test = sh(script: "curl http://${stack.Hostname}", returnStdout: true).trim()
                        echo test
                        // TODO test for string
                        input 'Test successful?'
                    } finally {
                        cfnDelete(stack: stackName)
                    }
                }
            }
        }
        stage('Notify') {
            echo "Build & Test IaC Demo version ${version} complete"
        }
    } catch (err) {
        echo "Build & Test IaC Demo version ${version} failed (${env.BUILD_URL})"
        throw err
    }
}
