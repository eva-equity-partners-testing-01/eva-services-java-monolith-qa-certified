pipeline {
    agent any

    parameters {

        string(
            name: 'PACKAGE_VERSION',
            defaultValue: '',
            description: 'Enter package version from CodeArtifact'
        )
    }

    environment {

        DEMO_HOST     = '35.175.126.147'
        DEMO_USER     = 'ubuntu'
        SSH_CRED_ID   = 'demo-environment'

        DEPLOY_DIR    = '/home/ubuntu/demo-environment'

        ARTIFACT_NAME = 'eva-services-java-monolith'
        GROUP_ID      = 'com.eva'
        REPO_NAME     = 'eva-services-java-monolith'
    }

    stages {

        stage('Deploy Selected Artifact') {

            steps {

                sshagent(credentials: [SSH_CRED_ID]) {

                    sh """
                    ssh -o StrictHostKeyChecking=no ${DEMO_USER}@${DEMO_HOST} '

                        set -e

                        cd ${DEPLOY_DIR}

                        echo "======================================"
                        echo "LOADING ENVIRONMENT"
                        echo "======================================"

                        source .env

                        export CODEARTIFACT_AUTH_TOKEN=\$(aws codeartifact get-authorization-token \
                          --domain eva-saas-domain \
                          --domain-owner 909783398453 \
                          --region us-east-1 \
                          --query authorizationToken \
                          --output text)

                        mkdir -p deploy
                        cd deploy

                        echo "======================================"
                        echo "DOWNLOADING PACKAGE"
                        echo "======================================"

                        mvn dependency:get \
                        -Dartifact=${GROUP_ID}:${ARTIFACT_NAME}:${params.PACKAGE_VERSION}:jar \
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
            echo "Package : com.eva:eva-services-java-monolith"
            echo "Version : ${params.PACKAGE_VERSION}"
            echo "Server  : ${DEMO_HOST}"
            echo "Path    : ${DEPLOY_DIR}"
            echo "=========================================="
        }

        failure {

            echo "=========================================="
            echo "DEPLOYMENT FAILED"
            echo "=========================================="
        }
    }
}
