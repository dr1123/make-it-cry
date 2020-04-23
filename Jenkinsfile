pipeline {
    agent any
    options { disableConcurrentBuilds() }
    environment {
        IMAGE_NAME="make-it-cry-dr"
        PORT="8999"
        IMAGE_ADDR="127.0.0.1:5000/${IMAGE_NAME}"
        VERSION_ID="${BUILD_ID}"
    }

    stages {
        stage('Test') {
            steps {
                sh 'chmod u+x mvnw'
                sh './mvnw clean test'
            }
        }
        stage('Build Image') {
            steps {
                sh '''
                ./mvnw package
                docker build -f Dockerfile -t ${IMAGE_NAME}:${VERSION_ID} .
                docker tag ${IMAGE_NAME}:${VERSION_ID} ${IMAGE_ADDR}:${VERSION_ID}
                docker push ${IMAGE_ADDR}:${VERSION_ID}
                docker rmi ${IMAGE_NAME}:${VERSION_ID}
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
                sh '''
                container_id=`docker ps|grep ${IMAGE_ADDR}|awk '{print $1}'`
                if [ -n "${container_id}" ]; then
                    docker rm -f "${container_id}"
                fi

                docker run -d -p ${PORT}:8080 ${IMAGE_ADDR}:${VERSION_ID}
                '''
            }
        }

    }
}
