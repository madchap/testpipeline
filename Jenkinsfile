@Library('prod-sec-libs') _

pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr:'15'))
        disableConcurrentBuilds()
    }
    environment {
        DOCKER_BASE_IMAGE = 'cloudbees/whitesource-agent'
        DOCKER_REPO_NAME = 'docker.io'
    }

    agent any
    stages {
        stage('Run Anchore') {
            steps {

                sh 'echo "${DOCKER_REPO_NAME}/${DOCKER_BASE_IMAGE}" > anchore_images'

                // artifacts automatically stored
                anchore name: 'anchore_images', forceAnalyze: false, bailOnFail: false, autoSubscribeTagUpdates: false, engineRetries: '300' // , policyBundleId: 'ewfwef'
                
            } 
            
            post {
                always {
                    script {
                        copyArtifacts(
                            fingerprintArtifacts: true,
                            flatten: true,
                            projectName: '${JOB_NAME}',
                            selector: specific('${BUILD_NUMBER}')
                        )
                        def engagementId = 3 // Defectdojo
                        echo "Uploading to DefectDojo engagementId $engagementId..."
                        withCredentials([string(credentialsId: 'dd-jenkins', variable: 'ddApiToken')]) {
                        testId = defectDojo.uploadScan_Anchore('anchoreengine-api-response-vulnerabilities-1.json', engagementId, env.ddApiToken)
                              echo "Test ID is $testId."
                        }
                    }
                }
            }
        }
    }
}


