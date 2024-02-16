@Library('pinVars') _  //se invoca libreria

def pinVarsInstance = pinVars() //se crea instancia para poder utilizar las funciones

pipeline {
  agent any

  options {
    timeout(time: 2, unit: 'MINUTES')
  }

  environment {
    VERSION_FILE = 'package.json'
  }

  stages {
    stage('Building image') {
      steps {
        script {
          try {
            echo "Extrayendo la versión de ${VERSION_FILE}"

            // utilizamos jq para extraer la versión
            def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()

            if (!version) {
              error 'No se encontró la versión en el package.json.'
            }

            echo "Versión encontrada en el package.json: ${version}"

            env.VERSION = version

            // Docker login
            
              sh '''
                ls -la
                docker build -t grupo7devops/pin1app:1.0.0 .
            '''
            

              
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

            
              pinVarsInstance.pushDockerImage()
           
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
