pipeline {
    agent any

    parameters {

        string(
            name: 'PACKAGE_VERSION',
            defaultValue: '',
            description: 'Which package version to deploy? Example: 1.0.0-QA-Peter-20260509061545'
        )
    }

    environment {

        DEMO_HOST     = '35.175.126.147'
        DEMO_USER     = 'ubuntu'
        SSH_CRED_ID   = 'demo-environment'

        DEPLOY_DIR    = '/home/ubuntu/demo-environment'

        MAVEN_HOME    = '/usr/share/maven'

        ARTIFACT_NAME = 'eva-services-java-monolith'
        GROUP_ID      = 'com.eva'
        REPO_NAME     = 'eva-services-java-monolith'
    }

    stages {

        stage('Deploy Selected Package') {

            steps {

                sshagent(credentials: [SSH_CRED_ID]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no ${DEMO_USER}@${DEMO_HOST} '

                        set -e

                        cd ${DEPLOY_DIR}

                        echo "=================================="
                        echo "Loading Environment"
                        echo "=================================="

                        source .env
                        
                        mkdir -p deploy
                        cd deploy

                        echo "=================================="
                        echo "Downloading Artifact"
                        echo "=================================="

                        mvn dependency:get \
                            -Dartifact=${GROUP_ID}:${ARTIFACT_NAME}:${PACKAGE_VERSION}:jar \
                            -DremoteRepositories=${REPO_NAME}::default::https://eva-saas-domain-909783398453.d.codeartifact.us-east-1.amazonaws.com/maven/${REPO_NAME}/ \
                            -DoutputDirectory=.
                    '
                    """
                }
            }
        }
    }

    post {

        success {
            echo "=========================================="
            echo "DEPLOYMENT SUCCESSFUL"
            echo "=========================================="
            echo "Server  : ${DEMO_HOST}"
            echo "Version : ${params.PACKAGE_VERSION}"
            echo "Path    : ${DEPLOY_DIR}"
            echo "=========================================="
        }

        failure {
            echo "DEPLOYMENT FAILED"
        }
    }
}
