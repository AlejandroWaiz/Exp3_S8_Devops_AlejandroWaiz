pipeline {
    agent any

    environment {
        REGISTRY      = 'docker.io'
        IMAGE_NAME    = 'tusuario/vehiculos-api'
        VERSION       = "v${BUILD_NUMBER}"
        DB_URL        = credentials('db_url')     
        DB_USER       = credentials('db_user')    
        DB_PASS       = credentials('db_pass')   
    }

    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Maven Build') { steps { sh './mvnw clean package -DskipTests' } }
        stage('Build Image') { steps { sh 'docker build -t $IMAGE_NAME:$VERSION .' } }
        stage('Publish Image') {
    steps {
        script {
            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                sh 'docker push $IMAGE_NAME:$VERSION'
                }
            }
        }
    }
        stage('Deploy') {
            steps {
                sh '''
                docker rm -f vehiculos || true
                docker run -d --name vehiculos \
                  -p 9090:8080 \
                  -e SPRING_DATASOURCE_URL=$DB_URL \
                  -e SPRING_DATASOURCE_USERNAME=$DB_USER \
                  -e SPRING_DATASOURCE_PASSWORD=$DB_PASS \
                  $IMAGE_NAME:$VERSION
                '''
            }
        }
    }
    post {
        success { echo '✔ Despliegue exitoso' }
        failure { echo '✖ Algo falló; revisa el log' }
    }
}
