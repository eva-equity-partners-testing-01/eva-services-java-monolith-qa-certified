pipeline {
    agent any

    parameters {

        choice(
            name: 'LABEL',
            choices: ['QA', 'Beta', 'Release'],
            description: 'Select Build Label'
        )

        string(
            name: 'VERSION',
            defaultValue: '1.0.0',
            description: 'Enter Version'
        )

        string(
            name: 'QA_PERSON',
            defaultValue: 'Peter',
            description: 'Enter QA Person Name'
        )
    }

    environment {

        QA_HOST        = '98.94.68.254'
        QA_USER        = 'ubuntu'
        SSH_CRED_ID    = 'qa-server-ssh-key'
        PROJECT_DIR    = '/home/ubuntu/QA-Certified/eva-services-java-monolith'
        MAVEN_HOME     = '/opt/apache-maven-3.5.2'
    }

    stages {

        stage('Prepare Build Metadata') {

            steps {

                script {

                    def buildDate = new Date().format("yyyyMMddHHmmss")

                    env.DYNAMIC_VERSION =
                        "${params.VERSION}-${params.LABEL}-${params.QA_PERSON}-${buildDate}"

                    env.DYNAMIC_VERSION =
                        env.DYNAMIC_VERSION.replaceAll("\\s+", "-")

                    echo "====================================="
                    echo "BUILD DETAILS"
                    echo "====================================="
                    echo "Label         : ${params.LABEL}"
                    echo "Version       : ${params.VERSION}"
                    echo "QA Person     : ${params.QA_PERSON}"
                    echo "Dynamic Build : ${env.DYNAMIC_VERSION}"
                    echo "====================================="
                }
            }
        }

        stage('Update pom.xml Version') {

            steps {

                sshagent(credentials: [SSH_CRED_ID]) {

                    sh """
                        ssh -o StrictHostKeyChecking=no ${QA_USER}@${QA_HOST} '

                            set -e

                            cd ${PROJECT_DIR}

                            echo "Updating pom.xml Version..."

                            sed -i "0,/<version>.*<\\\\/version>/s//<version>${DYNAMIC_VERSION}<\\\\/version>/" pom.xml

                            echo "Updated Version:"
                            grep "<version>" pom.xml | head -1
                        '
                    """
                }
            }
        }

        stage('Build & Publish Artifact') {

            steps {

                sshagent(credentials: [SSH_CRED_ID]) {

                    sh """
                        ssh -o StrictHostKeyChecking=no ${QA_USER}@${QA_HOST} '

                            set -e

                            cd ${PROJECT_DIR}

                            echo "====================================="
                            echo "LOADING ENV VARIABLES"
                            echo "====================================="

                            source .env

                            echo "====================================="
                            echo "JAVA VERSION"
                            echo "====================================="

                            java -version

                            echo "====================================="
                            echo "MAVEN VERSION"
                            echo "====================================="

                            ${MAVEN_HOME}/bin/mvn --version

                            echo "====================================="
                            echo "BUILDING & DEPLOYING ARTIFACT"
                            echo "====================================="

                            ${MAVEN_HOME}/bin/mvn clean deploy -DskipTests
                        '
                    """
                }
            }
        }

        stage('Store Build Metadata') {

            steps {

                writeFile file: 'build-info.txt', text: """
                BUILD INFORMATION
                ==============================
                
                Label           : ${params.LABEL}
                Version         : ${params.VERSION}
                QA Person       : ${params.QA_PERSON}
                Dynamic Version : ${env.DYNAMIC_VERSION}
                Build Time      : ${new Date()}
                
                """

                archiveArtifacts artifacts: 'build-info.txt', fingerprint: true

                echo "Build Metadata Stored Successfully"
            }
        }
    }

    post {

        success {

            echo "====================================="
            echo "ARTIFACT PUBLISHED SUCCESSFULLY"
            echo "====================================="
        }

        failure {

            echo "====================================="
            echo "BUILD FAILED"
            echo "====================================="
        }
    }
}
