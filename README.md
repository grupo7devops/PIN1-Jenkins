Proyecto PIN1 MundosE

Este proyecto utiliza Jenkins para automatizar el proceso de construcción, prueba y despliegue de una aplicación basada en Node.js.

Estructura del Proyecto
Jenkinsfile: El Jenkinsfile es un script declarativo que define el flujo de trabajo de Jenkins. Este archivo describe las etapas que Jenkins debe seguir para construir, probar y desplegar la aplicación.

pinVars.groovy: Este script Groovy es una biblioteca compartida que contiene funciones reutilizables para las acciones específicas de Docker, como el inicio de sesión, el build y el push de imágenes.

Dockerfile: El archivo Dockerfile describe la configuración de la imagen Docker utilizada para ejecutar la aplicación. En este proyecto, se basa en la imagen oficial de Node.js y realiza la configuración necesaria para instalar dependencias y exponer el puerto 3000.

package.json: Este archivo define las dependencias y scripts de la aplicación Node.js. También se utiliza para especificar la versión de la aplicación.

Jenkinsfile

Variables del Jenkinsfile
DOCKER_REGISTRY_URL: URL del registro de Docker donde se almacenarán las imágenes.
ARTIFACT_NAME: Nombre de la aplicación.
VERSION_FILE: Ruta del archivo que contiene la versión de la aplicación (en este caso, 'package.json').

environment {
    DOCKER_REGISTRY_URL = 'https://registry.example.com'
    ARTIFACT_NAME = 'webapp' 
    VERSION_FILE = 'package.json'
}


Stage 1: Construcción
La primera etapa (Build) se encarga de construir la aplicación. Aquí, se instalan las dependencias del proyecto y se realiza la construcción de la imagen Docker.

stage('Build') {
    steps {
        script {
            // Extraer la versión del archivo package.json
            def version = sh(script: "jq -r '.version' ${VERSION_FILE}", returnStdout: true).trim()
            
            // Construir la imagen Docker usando la función definida en pinVars.groovy
            def imageName = "${DOCKER_USERNAME}/${ARTIFACT_NAME}:${version}"
            pinVarsInstance.buildDockerImage(imageName)
        }
    }
}

Stage 2: Pruebas
La segunda etapa (Run Tests) ejecuta pruebas en la imagen Docker construida. La carpeta Test debe contener el archivo de prueba necesario

stage('Run Tests') {
    steps {
        script {
            // Ejecutar pruebas en la imagen Docker construida
            def imageName = "${DOCKER_USERNAME}/${ARTIFACT_NAME}:${version}"
            sh "docker run --rm ${imageName} npm test"
        }
    }
}

Stage 3: Despliegue
La tercera etapa (Deploy) se encarga de implementar la aplicación.

stage('Deploy') {
    steps {
        script {
            // Implementar la aplicación usando la función definida en pinVars.groovy
            def imageName = "${DOCKER_USERNAME}/${ARTIFACT_NAME}:${version}"
            pinVarsInstance.pushDockerImage(imageName)
        }
    }
}


pinVars.groovy
Este script Groovy contiene funciones reutilizables para las acciones específicas de Docker, como el inicio de sesión, el build y el push de imágenes.

def call() {
    def pinVars = [:]

    // Función para iniciar sesión en el registro de Docker
    pinVars.dockerLogin = { registryUrl, credentialsId ->
        withCredentials([usernamePassword(credentialsId: credentialsId, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
            withDockerRegistry([url: registryUrl]) {
                return true
            }
        }
        return false
    }

    // Función para construir la imagen Docker
    pinVars.buildDockerImage = { imageName ->
        sh "docker build -t ${imageName} ."
    }

    // Función para empujar la imagen Docker al registro
    pinVars.pushDockerImage = { imageName ->
        sh "docker push ${imageName}"
    }

    return pinVars
}
