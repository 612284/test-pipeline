properties([pipelineTriggers([githubPush()])])

pipeline {
   agent {
      label 'docker'
   }

    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerHub-user')		
		IMAGE='612284/kh-071-01-devops'
	}

    stages {
        stage('Checkout SCM on  GitHub') {
          steps {
            checkout([
              $class: 'GitSCM',
              branches: [[name: 'main']],
              userRemoteConfigs: [[
                url: 'https://github.com/612284/flask-app.git',
                credentialsId: '',
              ]]
             ])
           }
        }
        stage('Build docker image') {
            steps {
                dir('source') {
                  git branch: 'main', url: 'https://github.com/612284/flask-app.git'
                  script {
                    GIT_COMMIT_N = sh (
                      script: 'git rev-list --count main',
                      returnStdout: true
                    ).trim()
                    sh 'git rev-list --count main'
                 }
                  sh """docker build -t $IMAGE:${GIT_COMMIT_N}.${BUILD_NUMBER} ."""
                }
	        }
	    }
        stage('Push to Dockerhub'){
           steps{
             dir('source') {
               sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
               sh """docker push $IMAGE:${GIT_COMMIT_N}.${BUILD_NUMBER}"""
             }
	       }
	    }
        stage('Deploy to Prod'){
           steps{
              sshagent(['SSH-KEY']) {
                    script {
                        sh  'ssh -o StrictHostKeyChecking=no $USER@$PROD_IP uptime'
                        sh """ssh $USER@$PROD_IP sudo docker pull $IMAGE:${GIT_COMMIT_N}.${BUILD_NUMBER}"""
                        try {
                            sh """ssh $USER@$PROD_IP sudo docker run -d -p 80:5000 --name flask-app $IMAGE:${GIT_COMMIT_N}.${BUILD_NUMBER}"""
                            }
                        catch (exc) {
                             sh """
                              ssh $USER@$PROD_IP sudo docker stop flask-app
                              ssh $USER@$PROD_IP sudo docker rm flask-app
                              ssh $USER@$PROD_IP sudo docker run -d -p 80:5000 --name flask-app $IMAGE:${GIT_COMMIT_N}.${BUILD_NUMBER}
                             """
                            }
                        sh 'ssh $USER@$PROD_IP sudo docker ps'
                    }
              }
           }
        }
    }
    post {
		always {
			sh 'docker logout'
		}
	}
}
