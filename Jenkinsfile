pipeline {
    agent any

    parameters {
        string(
            name: 'PACKAGE_VERSION',
            defaultValue: '',
            description: 'Enter exact package version'
        )
    }

    environment {
        DEMO_HOST   = '35.175.126.147'
        DEMO_USER   = 'ubuntu'
        SSH_CRED_ID = 'demo-environment'

        DEPLOY_DIR = '/home/ubuntu/demo-environment'

        ARTIFACT_NAME = 'eva-services-java-monolith'
        GROUP_ID = 'com.eva'
        REPO_NAME = 'eva-services-java-monolith'
    }

    stages {

        stage('Deploy Artifact') {

            steps {

                sshagent(credentials: [SSH_CRED_ID]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no ${DEMO_USER}@${DEMO_HOST} '

                    set -e

                    cd ${DEPLOY_DIR}

                    rm -rf **

                    source .env
                    
                    mvn dependency:get \
                    -Dartifact=${GROUP_ID}:${ARTIFACT_NAME}:${params.PACKAGE_VERSION}:jar \
                    -DremoteRepositories=${REPO_NAME}::default::https://eva-saas-domain-909783398453.d.codeartifact.us-east-1.amazonaws.com/maven/${REPO_NAME}/ \
                    -DoutputDirectory=.

                    mv ~/.m2/repository/com/eva/${ARTIFACT_NAME}/${params.PACKAGE_VERSION} ${DEPLOY_DIR}/
                    
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
            echo "Version : ${params.PACKAGE_VERSION}"
            echo "Server  : ${DEMO_HOST}"
            echo "=========================================="
        }

        failure {
            echo "=========================================="
            echo "DEPLOYMENT FAILED"
            echo "=========================================="
        }
    }
}
