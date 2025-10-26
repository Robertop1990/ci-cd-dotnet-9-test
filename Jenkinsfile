pipeline {
    agent { label 'dotnet' }

    environment {
        CI_CONFIG = 'ci-cd.yml'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: readYaml(file: env.CI_CONFIG).git.branch,
                    credentialsId: readYaml(file: env.CI_CONFIG).git.credentialsId,
                    url: "https://github.com/Robertop1990/${readYaml(file: env.CI_CONFIG).project.name}.git"
            }
        }

        stage('Read Config') {
            steps {
                script {
                    config = readYaml file: env.CI_CONFIG
                    env.PROJECT_NAME = config.project.name
                    env.SOLUTION_FILE = config.project.solutionFile
                    env.PUBLISH_FOLDER = config.project.publishFolder
                    env.ZIP_NAME = config.project.zipName
                    env.BUILD_CONFIG = config.build.configuration
                    env.FRAMEWORK = config.build.framework
                    env.SONAR_URL = config.sonarqube.serverUrl
                    env.SONAR_CREDS = config.sonarqube.credentialsId
                    env.SONAR_PROJECT_KEY = config.sonarqube.projectKey
                    env.SONAR_PROJECT_NAME = config.sonarqube.projectName
                    env.SONAR_PROJECT_VERSION = config.sonarqube.projectVersion
                }
            }
        }

        stage('Debug Environment') {
            steps {
                script {
                    echo "===== Debug Variables ====="
                    echo "CI_CONFIG = ${env.CI_CONFIG}"
                    echo "PROJECT_NAME = ${env.PROJECT_NAME}"
                    echo "SOLUTION_FILE = ${env.SOLUTION_FILE}"
                    echo "PUBLISH_FOLDER = ${env.PUBLISH_FOLDER}"
                    echo "ZIP_NAME = ${env.ZIP_NAME}"
                    echo "BUILD_CONFIG = ${env.BUILD_CONFIG}"
                    echo "FRAMEWORK = ${env.FRAMEWORK}"
                    echo "SONAR_URL = ${env.SONAR_URL}"
                    echo "SONAR_CREDS = ${env.SONAR_CREDS}"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: env.SONAR_CREDS, variable: 'SONAR_TOKEN')]) {
                    script {
                        sh """
                            # Agregar herramientas globales al PATH
                            export PATH=\$PATH:/home/jenkins/.dotnet/tools

                            # Debug token length
                            echo "SONAR_TOKEN length: \$(echo \$SONAR_TOKEN | wc -c)"

                            # Ejecutar SonarScanner global
                            dotnet-sonarscanner begin \
                                /k:"${env.SONAR_PROJECT_KEY}" \
                                /n:"${env.SONAR_PROJECT_NAME}" \
                                /v:"${env.SONAR_PROJECT_VERSION}" \
                                /d:sonar.login=\$SONAR_TOKEN \
                                /d:sonar.host.url=${env.SONAR_URL}

                            dotnet build ${env.SOLUTION_FILE} -c ${env.BUILD_CONFIG} -f ${env.FRAMEWORK}

                            dotnet-sonarscanner end /d:sonar.login=\$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Publish & Zip') {
            steps {
                sh "dotnet publish ${env.SOLUTION_FILE} -c ${env.BUILD_CONFIG} -f ${env.FRAMEWORK} -o ${env.PUBLISH_FOLDER}"
                sh "cd ${env.PUBLISH_FOLDER} && zip -r ../${env.ZIP_NAME}.zip ."
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${env.ZIP_NAME}.zip", allowEmptyArchive: true
        }
    }
}