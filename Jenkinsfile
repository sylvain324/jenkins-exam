pipeline {
    agent any

    environment {
        DOCKER_HUB_ID = "lzmaurywehzexznjan"
        DOCKER_TAG = "v.0.${BUILD_ID}"
    }

    stages {
        stage("build") {
            parallel {
                stage("build cast-service") {
                    steps {
                        sh "docker build -t $DOCKER_HUB_ID/cast-service:$DOCKER_TAG cast-service"
                    }
                }
                stage("build movie-service") {
                    steps {
                        sh "docker build -t $DOCKER_HUB_ID/movie-service:$DOCKER_TAG movie-service"
                    }
                }
                stage("build proxy") {
                    steps {
                        sh "docker build -t $DOCKER_HUB_ID/proxy:$DOCKER_TAG proxy"
                    }
                }
            }
        }
        stage("Test") {
            steps {
                sh '''
                   docker compose up -d --wait
                   curl -s  http://localhost:8092/api/v1/movies/2/ > actual-result.json
                   diff tests/expected-result.json actual-result.json
                   docker compose down
                '''
            }
            post {
                always {
                    sh "docker compose down"
                }
            }
        }
        stage("Docker push") {
            environment {
                DOCKER_HUB_PASS = credentials("DOCKER_HUB_PASS")
            }
            steps {
                sh '''
                    docker login -u $DOCKER_HUB_ID -p $DOCKER_HUB_PASS
                    docker push -q $DOCKER_HUB_ID/cast-service:$DOCKER_TAG
                    docker push -q $DOCKER_HUB_ID/movie-service:$DOCKER_TAG
                    docker push -q $DOCKER_HUB_ID/proxy:$DOCKER_TAG
                '''
            }
        }

        stage("Deploiement en dev") {
            environment {
                KUBECONFIG = credentials("kubeconfig")
                ENVIRONNEMENT = "dev"
            }
            steps {
                sh '''
                  chmod 0600 $KUBECONFIG
                  helm upgrade --install v1.0 helm --namespace $ENVIRONNEMENT --set version="$DOCKER_TAG" --set namespace="$ENVIRONNEMENT" --set ingress_host="$ENVIRONNEMENT.mai23-devops.cloudns.ph"
                '''
            }
        }

        stage("Deploiement en qa") {
            environment {
                KUBECONFIG = credentials("kubeconfig")
                ENVIRONNEMENT = "qa"
            }
            steps {
                sh '''
                  chmod 0600 $KUBECONFIG
                  helm upgrade --install v1.0 helm --namespace $ENVIRONNEMENT --set version="$DOCKER_TAG" --set namespace="$ENVIRONNEMENT" --set ingress_host="$ENVIRONNEMENT.mai23-devops.cloudns.ph"
                '''
            }
        }

        stage("Deploiement en staging") {
            environment {
                KUBECONFIG = credentials("kubeconfig")
                ENVIRONNEMENT = "staging"
            }
            steps {
                sh '''
                  chmod 0600 $KUBECONFIG
                  helm upgrade --install v1.0 helm --namespace $ENVIRONNEMENT --set version="$DOCKER_TAG" --set namespace="$ENVIRONNEMENT" --set ingress_host="$ENVIRONNEMENT.mai23-devops.cloudns.ph"
                '''
            }
        }

        stage("Deploiement en prod") {
            environment {
                KUBECONFIG = credentials("kubeconfig")
                ENVIRONNEMENT = "prod"
            }
            steps {
                input 'Lancer le déploiement en production ?'
                sh '''
                  chmod 0600 $KUBECONFIG
                  helm upgrade --install v1.0 helm --namespace $ENVIRONNEMENT --set version="$DOCKER_TAG" --set namespace="$ENVIRONNEMENT" --set ingress_host="$ENVIRONNEMENT.mai23-devops.cloudns.ph"
                '''
            }
            when { branch "main" }
        }
    }
}
