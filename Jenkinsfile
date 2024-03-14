pipeline {
     environment {
       votre_id_dockerhub = "obichooooo"
       Votre_ID_GIT = "obichoo"
       IMAGE_NAME = "alpinehelloworld"
       IMAGE_TAG = "latest"
       PORT_EXPOSED = "8080"
       STAGING = "${votre_id_dockerhub}-staging"
       PRODUCTION = "${votre_id_dockerhub}-production"
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
                        rm -rf ${IMAGE_NAME}
                    
                    
                    # Cloner le dépôt
                    git clone https://github.com/${Votre_ID_GIT}/${IMAGE_NAME}.git
                    cd ${IMAGE_NAME}
                    
                    # Construire l'image Docker
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                  '''
                }
             }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    #!/bin/bash
                    # docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    # docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                    # sleep 5
                    
                    #!/bin/bash
                    docker build -t ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG} .
                    docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG}
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
                    curl http://172.17.0.1:${PORT_EXPOSED} | grep -q "Hello world!"
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

     stage ('Login and Push Image on docker hub') {
          agent any
        environment {
           DOCKERHUB_PASSWORD  = credentials('f00d20ee-8a7e-4db9-9d4e-023ade834bf9')
        }            
          steps {
             script {
               sh '''
                   echo ${DOCKERHUB_PASSWORD} | docker login -u ${votre_id_dockerhub} --password-stdin
                   docker push ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG}
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
}
