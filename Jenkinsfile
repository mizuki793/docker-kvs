pipeline {
  agent any
  environment {
    DOCKERHUB_USER = "fallakiakino"
    BUILD_HOST = "root@192.168.2.125"
    PROD_HOST = "root@192.168.2.126"
    BUILD_TIMESTAMP = sh(script: "date +%Y%m%d-%H%M%S", returnStdout: true).trim()
  }
  stages {
    stage('Pre Check') {
      steps {
        sh "test -f ~/.docker/config.json"
        sh "cat ~/.docker/config.json | grep docker.io"
      }
    }
    stage('Build') {
      steps {
        sh "cat docker-compose.build.yml"
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml down"
        sh "docker -H ssh://${BUILD_HOST} volume prune -f"
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml build"
        sh "COMPOSE_HTTP_TIMEOUT=600 docker-compose -H ssh://${ BUILD_HOST } -f docker-compose.build.yml up -d "
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml ps"
      }
    }
    stage('Test') {
      steps {
        sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_apptest pytest -v test_app.py"
        sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_static.py"
        sh "docker -H ssh://${BUILD_HOST} container exec dockerkvs_webtest pytest -v test_selenium.py"
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${BUILD_HOST} -f docker-compose.build.yml down"
      }
    }
    stage('Register') {
      steps {
        sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_web ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
        sh "docker -H ssh://${BUILD_HOST} tag dockerkvs_app ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
        sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_web:${BUILD_TIMESTAMP}"
        sh "docker -H ssh://${BUILD_HOST} push ${DOCKERHUB_USER}/dockerkvs_app:${BUILD_TIMESTAMP}"
      }
    }
    stage('Deploy') {
      steps {
        sh "cat docker-compose.prod.yml"
        sh "echo 'DOCKERHUB_USER=${DOCKERHUB_USER}' > .env"
        sh "echo 'BUILD_TIMESTAMP=${BUILD_TIMESTAMP}' >> .env"
        sh "cat .env"
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml up -d"
        sh "COMPOSE_HTTP_TIMEOUT=400 docker-compose -H ssh://${PROD_HOST} -f docker-compose.prod.yml ps"
      }
    }
  }
}