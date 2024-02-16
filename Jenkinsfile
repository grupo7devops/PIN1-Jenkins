@Library('pinVars') _

pipeline {
    agent any

    options {
        timeout(time: 2, unit: 'MINUTES')
    }

    environment {
        VERSION_FILE = 'package.json'
    }

    stages {
        stage('Build and Deploy') {
            steps {
                script {
                    try {
                        node { 
                            def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()

                            if (!version) {
                                error 'No se encontró la versión en el package.json.'
                            }

                            echo "Versión encontrada en el package.json: ${version}"

                            env.VERSION = version

                            // Docker login, build, and push
                            if (pinVars.dockerLogin('https://registry.example.com')) {
                                pinVars.buildAndPushDockerImage("${DOCKER_USER}/pin1app", "${version}", '.')
                            }
                        } // fin bloque node
                    } catch (Exception e) {
                        echo "Error en la etapa de Build y Deploy: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error 'Hubo un error durante la etapa de Build y Deploy.'
                    }
                }
            }
        }
    }
}
