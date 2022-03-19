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
    }
    post {
		always {
			sh 'docker logout'
		}
	}
}
