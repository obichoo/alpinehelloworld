@Library('shared-library')_
pipeline {
  environment {
    PROD_APP_ENDPOINT = "https://obichooooo-staging-b1f04106fc86.herokuapp.com/"
    STG_APP_ENDPOINT = "https://obichooooo-staging-b1f04106fc86.herokuapp.com/"
    ID_DOCKERHUB = 'obichooooo'
    ID_GIT = 'obichoo'
    IMAGE_NAME = 'ci-cd-projet-1'
    IMAGE_TAG = 'latest'
    PORT_EXPOSED = '5000'
    STAGING = "${ID_DOCKERHUB}-staging"
    PRODUCTION = "${ID_DOCKERHUB}-production"
  }
  agent none
  stages {
    stage('Build image') {
      agent any
      steps {
        script {
          sh '''
                   #!/bin/bash
                    # Nettoyer le répertoire s'il existe déjà
                    if [ -d "${IMAGE_NAME}" ]; then
                        rm -rf ${IMAGE_NAME}
                    fi

                    # Cloner le dépôt
                    git clone https://github.com/${ID_GIT}/${IMAGE_NAME}.git
                    cd ${IMAGE_NAME}

                    docker build -t ${ID_DOCKERHUB}/${IMAGE_NAME}:${IMAGE_TAG} --platform linux/amd64 .
                  '''
        }
      }
    }
        stage('Run container based on builded image') {
      agent any
      steps {
        script {
          sh '''
                    docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${ID_DOCKERHUB}/${IMAGE_NAME}:${IMAGE_TAG}
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
                    curl http://172.17.0.1:80 | grep -q "Hello world!"
                '''
        }
      }
    }
      stage('Clean Container') {
          agent any
          steps {
        script {
          sh '''
                 docker stop ${IMAGE_NAME}
                 docker rm ${IMAGE_NAME}
               '''
        }
          }
      }

    stage('Login and Push Image on docker hub') {
          agent any
        environment {
        DOCKERHUB_CREDENTIALS  = credentials('DOCKERHUB_LOGS')
        }
          steps {
        script {
          sh '''
                   echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                   docker push ${ID_DOCKERHUB}/${IMAGE_NAME}:${IMAGE_TAG}
               '''
        }
          }
    }

    stage('Push image in staging and deploy it') {
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
              npm i -g heroku@7.68.0
              heroku container:login
              heroku create $STAGING || echo "project already exist"
              heroku container:push -a $STAGING web
              heroku container:release -a $STAGING web
            '''
          }
      }
    }

    stage('Push image in production and deploy it') {
      when {
        expression { GIT_BRANCH == 'origin/production' }
      }
      agent any
      environment {
        HEROKU_API_KEY = credentials('heroku_api_key')
      }
      steps {
        script {
          sh '''
            npm i -g heroku@7.68.0
            heroku container:login
            heroku create $PRODUCTION || echo "project already exist"
            heroku container:push -a $PRODUCTION web
            heroku container:release -a $PRODUCTION web
          '''
        }
      }
    }
  }
  post {
    always {
      script {
        slackNotifier currentBuild.result
      }
    }  
  }
}
