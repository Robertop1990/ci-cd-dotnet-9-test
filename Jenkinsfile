pipeline {
    agent { label 'dotnet' } // El agente que creamos en Docker

    environment {
        CI_CONFIG = 'ci-cd.yml' // Ruta del archivo YAML en el repo
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

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: env.SONAR_CREDS, variable: 'SONAR_TOKEN')]) {
                    sh """
                        dotnet sonarscanner begin \
                        /k:"${env.SONAR_PROJECT_KEY}" \
                        /n:"${env.SONAR_PROJECT_NAME}" \
                        /v:"${env.SONAR_PROJECT_VERSION}" \
                        /d:sonar.login=$SONAR_TOKEN \
                        /d:sonar.host.url=${env.SONAR_URL}
                    """
                    sh "dotnet build ${env.SOLUTION_FILE} -c ${env.BUILD_CONFIG} -f ${env.FRAMEWORK}"
                    sh "dotnet sonarscanner end /d:sonar.login=$SONAR_TOKEN"
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
