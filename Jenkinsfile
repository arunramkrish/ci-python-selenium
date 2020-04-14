pipeline {
    agent any

    triggers {
        pollSCM('*/5 * * * 1-5')
    }

    options {
        skipDefaultCheckout(true)
        // Keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        stage ("Code pull"){
            steps{
                checkout scm
            }
        }

        stage('Unit tests and acceptance tests') {
            steps {
                //sh  ''' source activate ${BUILD_TAG}
                //        python -m pytest --verbose --junit-xml reports/unit_tests.xml
                //    '''
                bat "python -m pytest --verbose --junit-xml reports/unit_tests.xml tests/login_scenario.py"
                bat "behave"
            }
            post {
                always {
                    // Archive unit tests for the future
                    echo "Done building"
                    junit allowEmptyResults: true, testResults: 'reports/unit_tests.xml'
                }
                success {
                    echo "Successful"
                }
                failure {
                    emailext (
                        subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                        body: """<p>FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
                         <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
                        recipientProviders: [[$class: 'CulpritsRecipientProvider']]
                    )
                }
            }
        }
    }

}