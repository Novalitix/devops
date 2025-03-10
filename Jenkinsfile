pipeline {
    agent any

    environment {
        REGISTRY = 'florian06'
        DOCKER_CREDENTIALS_ID = 'Novalitix_Github_Token' // Utiliser votre identifiant réel Jenkins ici
        VPS_CREDENTIALS_SSH_ID = 'VPS_CREDENTIALS_SSH_ID'       // Utiliser votre identifiant réel Jenkins ici
        VPS_HOST = '37.27.13.174'
        DOCKER_COMPOSE_PATH = '/root/devops/devops/mlo-ai'
        GITHUB_CLE = 'GITHUB_CLE'
        IMAGE_BACKENG = 'mlo_backend'
        IMAGE_FRONTEND = 'mlo_frontend'
        CONTAINER_BACKENG = 'mlo-backend'
        CONTAINER_FRONTEND = 'mlo-frontend'
        NAME_BRANCH = 'develop'
        VERSION = '1.1'
        URL_GITHUB_BACKENG = 'https://github.com/Novalitix/mlo-ai-backend.git'
        URL_GITHUB_FRONTEND = 'https://github.com/Novalitix/mlo-ai-frontend.git'
    }

    stages {
        stage('Checkout Backend') {
            steps {
                script {
                    echo "Clonage du dépôt Backend avec la clé d'identification..."
                    try {
                        checkoutFromRepo(URL_GITHUB_BACKENG, NAME_BRANCH, GITHUB_CLE)
                    } catch (Exception e) {
                        echo "Erreur lors du clonage du dépôt Backend : ${e.message}"
                        error("Échec de l'étape de Checkout du Backend")
                    }
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                script {
                    echo "Déploiement du service Backend..."
                    try {
                        deployDockerService(IMAGE_BACKENG, CONTAINER_BACKENG, VERSION)
                    } catch (Exception e) {
                        echo "Erreur lors du déploiement Backend : ${e.message}"
                        error("Échec du déploiement du service Backend")
                    }
                }
            }
        }


        stage('Checkout Frontend') {
            steps {
                script {
                    echo "Clonage du dépôt Frontend avec la clé d'identification..."
                    try {
                        checkoutFromRepo(URL_GITHUB_FRONTEND, NAME_BRANCH, GITHUB_CLE)
                    } catch (Exception e) {
                        echo "Erreur lors du clonage du dépôt Frontend : ${e.message}"
                        error("Échec de l'étape de Checkout du Frontend")
                    }
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                script {
                    echo "Déploiement du service Frontend..."
                    try {
                        deployDockerService(IMAGE_FRONTEND, CONTAINER_FRONTEND, VERSION)
                    } catch (Exception e) {
                        echo "Erreur lors du déploiement Frontend : ${e.message}"
                        error("Échec du déploiement du service Frontend")
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Nettoyage du workspace..."
                if (currentBuild.result == 'FAILURE') {
                    echo "Le build a échoué, vérifiez les logs pour plus de détails."
                }
                cleanWs()
            }
        }
    }
}

def deployDockerService(serviceName, serviceNameContainer, version) {
    echo "Construction de l'image Docker pour ${serviceName} avec la version ${version}..."
    sh "docker build -t ${REGISTRY}/${serviceName}:${version} ."

    // withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
    //     echo "Connexion au registre Docker avec les identifiants ${DOCKER_CREDENTIALS_ID}..."
    //     sh "echo \$DOCKER_PASSWORD | docker login ${REGISTRY} --username \$DOCKER_USERNAME --password-stdin"
    // }

    echo "Poussée de l'image ${serviceName}:${version} vers le registre Docker..."
    sh "docker push ${REGISTRY}/${serviceName}:${version}"
    echo "Poussée de l'image ${serviceName}:${version} vers le registre Docker OK OK"


    // Determine the correct service name
    def serviceNameCorrect = sh(
        script: """
            if [ "${serviceName}" = "sgt_backend" ]; then
                echo "sgt-backend";
            elif [ "${serviceName}" = "sgt_frontend" ]; then
                echo "sgt-frontend";
            else
                echo "${serviceName}";
            fi
        """,
        returnStdout: true
    ).trim()

    echo "Correct service name: ${serviceNameContainer}"

    // Connect to the VPS and update the service
    sshagent([VPS_CREDENTIALS_SSH_ID]) {
        echo "Connecting to VPS and updating service ${serviceNameContainer} on ${VPS_HOST}..."
        sh """
            ssh -o StrictHostKeyChecking=no root@${VPS_HOST} '
                cd ${DOCKER_COMPOSE_PATH} &&

                if [ \$(docker ps -a -q -f name=${serviceNameContainer}) ]; then
                    echo "Removing container ${serviceNameContainer}..." &&
                    docker rm -f ${serviceNameContainer};
                fi

                if [ \$(docker images -q ${REGISTRY}/${serviceName}:${version}) ]; then
                    echo "Removing image ${REGISTRY}/${serviceName}:${version}..." &&
                    docker rmi ${REGISTRY}/${serviceName}:${version};
                fi

                docker compose -f docker-compose.bd.prod.yml -f docker-compose.prod.yml up -d &&
                docker logs ${serviceNameContainer}
            '
        """
    }
}

def checkoutFromRepo(repoUrl, branchName, credentialsId) {
    echo "Clonage du dépôt ${repoUrl} sur la branche ${branchName} avec les credentials ${credentialsId}..."
    git branch: branchName,
        credentialsId: credentialsId,
        url: repoUrl
}



