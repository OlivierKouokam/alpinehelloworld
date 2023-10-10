pipeline {
  environment {
      IMAGE_NAME = "alpinehelloworld"
      APP_EXPOSED_PORT = "8090"
      APP_NAME = "jenkins-lab"
      IMAGE_TAG = "latest"
      STAGING = "${APP_NAME}-staging"
      PRODUCTION = "${APP_NAME}-prod"
      DOCKERHUB_ID = "olivierkkoc"
      // DOCKERHUB_PASSWORD = credentials('dockerhub')
      DOCKERHUB_PASSWORD_USR = "olivierkkoc"
      DOCKERHUB_PASSWORD_PSW = "Godislove/*-"
      STG_API_ENDPOINT = "ip10-0-0-3-ckiok9ct654gqaevksdg-1993.direct.docker.labs.eazytraining.fr"
      STG_APP_ENDPOINT = "ip10-0-0-3-ckiok9ct654gqaevksdg-80.direct.docker.labs.eazytraining.fr"
      PROD_API_ENDPOINT = "ip10-0-0-4-ckiok9ct654gqaevksdg-1993.direct.docker.labs.eazytraining.fr"
      PROD_APP_ENDPOINT = "ip10-0-0-4-ckiok9ct654gqaevksdg-80.direct.docker.labs.eazytraining.fr"
      INTERNAL_PORT = "5000"
      EXTERNAL_PORT = "$APP_EXPOSED_PORT"
      CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

  agent none

  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          sh 'docker build -t $DOCKERHUB_ID/$IMAGE_NAME:$IMAGE_TAG .'
        }
      }
    }

    stage('Run container based on builded image') {
      agent any
      steps {
        script {
          sh '''
            echo "Cleaning existing container if exist"
            docker ps -a | grep -i $IMAGE_NAME && docker stop $IMAGE_NAME
            docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
            docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$INTERNAL_PORT -e PORT=$INTERNAL_PORT $DOCKERHUB_ID/$IMAGE_NAME:$IMAGE_TAG
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
            curl -v http://172.17.0.1:$APP_EXPOSED_PORT | grep -q "Hello world!"            
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

    stage ('Login and Push Image on docker hub') {
        agent any
        steps {
            script {
              sh '''
                  echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
                  docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
              '''
            }
        }
    }

    stage('STAGING env - Deploy app ') {
      when {
        expression { GIT_BRANCH == 'origin/eazylabs' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh """
            echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
            curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
          """
        }
      }
    }

    stage('Push image in production and deploy it ') {
      when {
        expression { GIT_BRANCH == 'origin/master' }
      }
      agent any
      steps {
        script {
          sh """
            echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json 
            curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
          """
        }
      }
    }
  }
}
