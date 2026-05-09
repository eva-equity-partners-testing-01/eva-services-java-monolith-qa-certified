pipeline {
    agent any

    parameters {

        choice(
            name: 'LABEL',
            choices: ['QA', 'Beta', 'Release'],
            description: 'Select build type for artifact publishing'
        )
    
        string(
            name: 'VERSION',
            defaultValue: '',
            description: 'Enter release version (Example: 1.0.0)'
        )
    
        string(
            name: 'QA_PERSON',
            defaultValue: '',
            description: 'Enter QA engineer name'
        )
    }

    environment {
        QA_HOST     = '98.94.68.254'
        QA_USER     = 'ubuntu'
        SSH_CRED_ID = 'qa-server-ssh-key'
        PROJECT_DIR = '/home/ubuntu/QA-Certified/eva-services-java-monolith'
        MAVEN_HOME  = '/opt/apache-maven-3.5.2'
    }

    stages {

        stage('Prepare Metadata') {
            steps {
                script {
                    def buildDate = new Date().format("yyyyMMddHHmmss")

                    env.DYNAMIC_VERSION =
                        "${params.VERSION}-${params.LABEL}-${params.QA_PERSON}-${buildDate}"
                        .replaceAll("\\s+","-")

                    echo env.DYNAMIC_VERSION
                }
            }
        }

        stage('Update Version') {
            steps {
                sshagent(credentials: [SSH_CRED_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${QA_USER}@${QA_HOST} '
                    cd ${PROJECT_DIR}

                    sed -i "/<artifactId>eva-services-java-monolith<\\\\/artifactId>/{n;s#<version>.*</version>#<version>${DYNAMIC_VERSION}</version>#;}" pom.xml
                    '
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: [SSH_CRED_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${QA_USER}@${QA_HOST} '
                    cd ${PROJECT_DIR}

                    source .env

                    ${MAVEN_HOME}/bin/mvn clean deploy -DskipTests
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "=========================================="
            echo "ARTIFACT PUBLISHED SUCCESSFULLY"
            echo "=========================================="
            echo "Package Name : com.eva:eva-services-java-monolith"
            echo "Version      : ${env.DYNAMIC_VERSION}"
            echo "Published To : AWS CodeArtifact"
            echo "=========================================="

        }

        failure {
            echo "DEPLOY FAILED"
        }
    }
}
