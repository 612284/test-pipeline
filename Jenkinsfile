pipeline {
    agent any
    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerHub-user')
		USER='ubuntu'		
		IMAGE='612284/spring-petclinic'
	}

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
    }
    stage('Push to Dockerhub'){
        steps{
           dir('source') {
             sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
             echo 'login ok'
           }
        }
    }        
    post {
		always {
			sh 'docker logout'
		}
	}
}
