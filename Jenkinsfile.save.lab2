pipeline {
  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "latest"
    STAGING = "eazytraining-staging"
    PRODUCTION = "eazytraining-production"
  }

  agent none

  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          sh 'docker build -t olivierkkoc/$IMAGE_NAME:$IMAGE_TAG .'
        }
      }
    }

    stage('Run container based on builded image') {
      agent any
      steps {
        script {
          sh '''
            docker run --name $IMAGE_NAME -d -p 8090:5000 -e PORT=5000 olivierkkoc/$IMAGE_NAME:$IMAGE_TAG
            sleep 5
          '''
        }
      }
    }

    stage('Test image') {
      agent any
      steps {
        script {
          sh '''
            curl http://172.17.0.1:8090 | grep -q "Hello world!"            
          '''
        }
      }
    }

    stage('Clean Container') {
      agent any
      steps {
        script {
          sh '''
            docker stop $IMAGE_NAME
            docker rm $IMAGE_NAME
          '''
        }
      }
    }

    stage('Push image in staging and deploy it ') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh '''
            echo "push image - staging"
            echo "deploy image in staging env"
          '''
        }
      }
    }

    stage('Push image in production and deploy it ') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh '''
            echo "push image - production"
            echo "deploy image in production env"
          '''
        }
      }
    }
  }
}
