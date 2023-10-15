pipeline {
    agent any

    environment {
        DOCKER_HUB_ID = "lzmaurywehzexznjan"
        DOCKER_TAG = "v.0.${BUILD_ID}"
    }

    stages {
        stage("build") {
            steps {
                sh '''
                  docker build -t $DOCKER_HUB_ID/cast-service:$DOCKER_TAG cast-service
                  docker build -t $DOCKER_HUB_ID/movie-service:$DOCKER_TAG movie-service
                  docker build -t $DOCKER_HUB_ID/proxy:$DOCKER_TAG proxy
                '''
            }
        }
    }
}
