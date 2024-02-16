@Library('pinVars') _

def pinVarsInstance = pinVars()

pipeline {
    agent any

    options {
        timeout(time: 2, unit: 'MINUTES')
    }

    environment {
        DOCKER_REGISTRY_URL = 'https://registry.example.com'
        ARTIFACT_NAME = 'pin1app'
        VERSION_FILE = 'package.json'
    }

    stages {
        stage('Docker Login') {
            steps {
                script {
                    pinVarsInstance.dockerLogin(DOCKER_REGISTRY_URL, 'dockerHub')
                }
            }
        }

        stage('Building image') {
            steps {
                script {
                    try {
                        echo "Extrayendo la versión de ${VERSION_FILE}"

                        def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()

                        if (!version) {
                            error 'No se encontró la versión en el package.json.'
                        }

                        echo "Versión encontrada en el package.json: ${version}"

                        env.ARTIFACT_VERSION = version

                        // Componer el nombre de la imagen usando DOCKER_USER de Jenkins
                        def imageName = "${DOCKER_USER}/${ARTIFACT_NAME}:${env.ARTIFACT_VERSION}"
                        pinVarsInstance.buildDockerImage(imageName)
                    } catch (Exception e) {
                        echo "Error en la etapa de Build: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error 'Hubo un error durante la etapa de Build.'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Componer el nombre de la imagen usando DOCKER_USER de Jenkins
                    def imageName = "${DOCKER_USER}/${ARTIFACT_NAME}:${env.ARTIFACT_VERSION}"
                    pinVarsInstance.pushDockerImage(imageName)
                }
            }
        }
    }
}
