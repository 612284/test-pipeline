properties([pipelineTriggers([githubPush()])])

pipeline {
   agent {
      label 'docker'
   }

    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerHub-user')
		USER='ubuntu'
		IMAGE='612284/kh-071-01-devops'
	}

    stages {
        stage('Checkout SCM') {
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
        stage('build') {
            steps {
                dir('source') {
                  git branch: 'main', url: 'https://github.com/612284/flask-app.git'
                  sh 'docker build -t $IMAGE:${BUILD_NUMBER} .'
                }
	        }
	    }
        stage('Push to Dockerhub'){
           steps{
             dir('source') {
               sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
               sh 'docker push $IMAGE:${BUILD_NUMBER}'
             }
	       }
	    }
        stage('Deploy'){
           steps{
              sshagent(['worker-SSH-KEY']) {
                    script {
                        sh  'ssh -o StrictHostKeyChecking=no $USER@$PROD_IP uptime'
                        sh """ssh $USER@$PROD_IP sudo docker pull $IMAGE:${BUILD_NUMBER}"""
                        try {
                            sh """ssh $USER@$PROD_IP sudo docker run -d -p 80:5000 --name flask-app $IMAGE:${BUILD_NUMBER}"""
                            }
                        catch (exc) {
                             sh """
                              ssh $USER@$PROD_IP sudo docker stop flask-app
                              ssh $USER@$PROD_IP sudo docker rm flask-app
                              ssh $USER@$PROD_IP sudo docker run -d -p 80:5000 --name flask-app $IMAGE:${BUILD_NUMBER}
                             """
                            }
                        sh """ssh $USER@$PROD_IP sudo docker ps"""
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
