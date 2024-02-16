@Library('pinVars') _  //se invoca libreria

def pinVarsInstance = pinVars() //se crea instancia para poder utilizar las funciones

pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
    VERSION_FILE = 'package.json'
    DOCKER_USER = credentials('dockerHub')
  }

  stages {
    stage('Building image') {
      steps {
        script {
          try {
            echo "Extrayendo la versión de ${VERSION_FILE}"

            // traza de depuración
            def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()
            echo "Versión encontrada en el package.json: ${version}"

            // utilizamos jq para extraer la versión
            def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()

            if (!version) {
              error 'No se encontró la versión en el package.json.'
            }

            echo "Versión encontrada en el package.json: ${version}"

            env.VERSION = version

            //traza de depuración
            echo "DOCKER_USER: ${DOCKER_USER}"
            echo "VERSION: ${VERSION}"

            // traza de depuración para la construcción de la imagen
            sh """
                            ls -la
                            docker build -t ${DOCKER_USER}/pin1app:${version} .
                        """

              // Docker login
              if (pinVarsInstance.dockerLogin('https://registry-1.docker.io/v2/')) {
              //traza de depuración para la etapa de Deploy
              echo "Realizando deploy con DOCKER_USER: ${DOCKER_USER}, VERSION: ${env.VERSION}"
              pinVarsInstance.buildDockerImage("${DOCKER_USER}/pin1app", "${version}")
              }
          }catch (Exception e) {
            echo "Error en la etapa de Build: ${e.message}"
            currentBuild.result = 'FAILURE'
            error 'Hubo un error durante la etapa de Build.'
          }
        }
      }
    }

    // stage('Run tests') {
    //   steps {
    //     sh "docker run testapp npm test"
    //   }
    // }

    stage('Deploy') {
      steps {
        script {
          try {
            // Docker logindef loggedIn = pinVarsInstance.dockerLogin('https://registry.example.com')

            if (pinVarsInstance.dockerLogin('https://registry.example.com')) {
              pinVarsInstance.pushDockerImage("${DOCKER_USER}/pinapp", "${env.VERSION}", '.')
            }
                    } catch (Exception e) {
            echo "Error en la etapa de Deploy: ${e.message}"
            currentBuild.result = 'FAILURE'
            error 'Hubo un error durante la etapa de Deploy.'
          }
        }
      }
        } // fin stage deploy
  }
}
