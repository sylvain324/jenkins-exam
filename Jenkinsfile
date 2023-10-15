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
            }
            steps {
                echo "$KUBECONFIG"
            }
        }
    }

    post {
        always {
            sh "docker compose down"
        }
    }
}
