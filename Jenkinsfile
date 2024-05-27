/* import shared library */
@Library('Bah27-share-library')_

pipeline {
    environment {
        IMAGE_NAME = "site_wordpress"
        IMAGE_TAG = "latest"
        STAGING = "mokrane921-staging"
        PRODUCTION = "mokrane921-production"
    }
    agent none
    stages {
       stage('Build image') {
           agent {
               docker { 
                   image 'docker:latest' 
                   args '-v /var/run/docker.sock:/var/run/docker.sock'
               }
           }
           steps {
              script {
                sh "docker build -t mokrane921/${IMAGE_NAME}:${IMAGE_TAG} ."
              }
           }
       }
       stage('Run container based on built image') {
          agent {
              docker { 
                  image 'docker:latest' 
                  args '-v /var/run/docker.sock:/var/run/docker.sock'
              }
          }
          steps {
            script {
              sh """
                  docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT=5000 mokrane921/${IMAGE_NAME}:${IMAGE_TAG}
                  sleep 5
              """
             }
          }
       }
       stage('Test image') {
           agent any
           steps {
              script {
                sh """
                   curl localhost:80 | grep "Hello world!"
                """
              }
           }
       }
       stage('Clean container') {
          agent {
              docker { 
                  image 'docker:latest' 
                  args '-v /var/run/docker.sock:/var/run/docker.sock'
              }
          }
          steps {
             script {
               sh """
                   docker stop ${IMAGE_NAME}
                   docker rm ${IMAGE_NAME}
               """
             }
          }
      }
      stage('Push image to staging and deploy it') {
        when {
            branch 'main'
        }
        agent any
        environment {
            HEROKU_API_KEY = credentials('heroku_api_key')
        }
        steps {
           script {
             sh """
                heroku container:login
                heroku create ${STAGING} || echo "projects already exist"
                heroku container:push web -a ${STAGING}
                heroku container:release web -a ${STAGING}
             """
           }
        }
     }
     stage('Push image to production and deploy it') {
       when {
           branch 'main'
       }
       agent any
       environment {
           HEROKU_API_KEY = credentials('heroku_api_key')
       }
       steps {
          script {
            sh """
               heroku container:login
               heroku create ${PRODUCTION} || echo "projects already exist"
               heroku container:push web -a ${PRODUCTION}
               heroku container:release web -a ${PRODUCTION}
            """
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
