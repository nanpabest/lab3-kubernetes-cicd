pipeline {

    agent {
        kubernetes {
            yamlFile 'agent.yaml'
            retries 2
        }
    }

    environment {
        APP_VERSION    = '3.0.0'
        DOCKER_IMAGE   = 'hernancontreras/tarea-final'
        NAME_TAG       = 'hernan-contreras'

        K8S_NAMESPACE  = 'ns-hernan-contreras'
        K8S_DEPLOYMENT = 'app-hernan-contreras'
        K8S_CONTAINER  = 'app-hernan-contreras'
    }

    stages {

        stage('install') {
            steps {
                container('node') {
                    sh '''
                        corepack enable
                        pnpm install --frozen-lockfile
                    '''
                }
            }
        }

        stage('test') {
            steps {
                container('node') {
                    sh '''
                        pnpm test -- --runInBand
                    '''
                }
            }
        }

        stage('build') {
            steps {
                container('docker') {
                    sh '''
                        echo "Esperando Docker daemon..."

                        until docker info > /dev/null 2>&1
                        do
                            sleep 2
                        done

                        docker build \
                            -t ${DOCKER_IMAGE}:${APP_VERSION} \
                            -t ${DOCKER_IMAGE}:${NAME_TAG} \
                            .
                    '''
                }
            }
        }

        stage('push') {
            steps {
                container('docker') {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'dockerhub-credentials',
                            usernameVariable: 'DOCKERHUB_USER',
                            passwordVariable: 'DOCKERHUB_TOKEN'
                        )
                    ]) {
                        sh '''
                            echo "$DOCKERHUB_TOKEN" | docker login \
                                -u "$DOCKERHUB_USER" \
                                --password-stdin

                            docker push ${DOCKER_IMAGE}:${APP_VERSION}
                            docker push ${DOCKER_IMAGE}:${NAME_TAG}

                            docker logout
                        '''
                    }
                }
            }
        }

        stage('deploy') {
            steps {
                container('kubectl') {
                    sh '''
                        kubectl set image \
                            deployment/${K8S_DEPLOYMENT} \
                            ${K8S_CONTAINER}=${DOCKER_IMAGE}:${NAME_TAG} \
                            -n ${K8S_NAMESPACE}

                        kubectl rollout restart \
                            deployment/${K8S_DEPLOYMENT} \
                            -n ${K8S_NAMESPACE}

                        kubectl rollout status \
                            deployment/${K8S_DEPLOYMENT} \
                            -n ${K8S_NAMESPACE} \
                            --timeout=180s

                        kubectl get pods \
                            -n ${K8S_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline CI/CD finalizado correctamente.'
        }

        failure {
            echo 'Pipeline CI/CD finalizado con error.'
        }
    }
}