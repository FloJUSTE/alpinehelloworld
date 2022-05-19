pipeline {

  environment {
    IMAGE_NAME = "alpinehelloworld"
    IMAGE_TAG = "latest"
    ID_DOCKER = "papaflo"
    DOCKER_CRED = credentials('dockerhub_pwd')
    STAGING = "${ID_DOCKER}-staging"
    PRODUCTION = "${ID_DOCKER}-production"
  }
  
  agent none
  
  stages {
    stage('Création image') {
      agent any
      steps {
        script {
          sh 'docker build -t $ID_DOCKER/$IMAGE_NAME:IMAGE_TAG .'
        }
      }
    }
    
    stage('Création du conteneur') {
      agent any
      steps {
        script {
          sh '''
            docker run -d -p 80:5000 -e PORT=5000 --name $IMAGE_NAME $ID_DOCKER/$IMAGE_NAME:$IMAGE_TAG 
            sleep 5
          '''
        }
      }
    }
	
    
    stage('Test image') {
      agent any
      steps {
        script {
          sh 'curl http://172.17.0.1 | grep -q "Hello world!"'
        }
      }
    }
    
    stage('Clean Container') {
       agent any
       steps {
         script {
           sh '''
              docker stop $IMAGE_NAME
	            docker rm  $IMAGE_NAME
           '''
         }
       }
    }
     
    stage('Login and Push image sur Docker Hub') {
      agent any
      steps {
        script {
          sh '''
            docker login -u=$ID_DOCKER -p=$DOCKER_CRED
            docker push $ID_DOCKER/$IMAGE_NAME
          '''
        }
      }
    }
  
    stage('Pousser image sur HEROKU en STAGING') {
      when {
        expression { GIT_BRanCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_KEY = credentials('heroku')
      }
      steps {
        script {
          sh '''
            heroku container:login
            heroku create $STAGING || echo "project already exist"
            heroku container:push -a $STAGING web
            heroku container:release -a $STAGING web
          '''
        }
      }
    }
    
    stage('Pousser image sur HEROKU en PRODUCTION') {
      when {
        expression { GIT_BRanCH == 'origin/master' }
      }
      agent any
      environment {
        HEROKU_KEY = credentials('heroku')
      }
      steps {
        script {
          sh '''
            heroku container:login
            heroku create $STAGING || echo "project already exist"
            heroku container:push -a $PRODUCTION web
            heroku container:release -a $PRODUCTION web
          '''
        }
      }
    }
  }
}
  

