pipeline{
    agent any
    environment {
        PATH = "$PATH:/usr/share/apache-maven/bin"
        DOCKERHUB_CREDENTIALS = credentials('dockerhubtoken')
    }
    stages{
       stage('GetCode'){
            steps{
                git url: 'https://github.com/puneetgavri/java-web-app-docker-demo.git'
            }
       }
       stage('debuging'){
            steps{
                sh 'java -version'
                sh 'mvn --version'
            }
       }
       stage('Build'){
            steps{
                sh 'mvn clean package'
            }
       }
       stage('SonarQube analysis') {
            steps{
                  withSonarQubeEnv('mysonarserver') { 
                  sh "mvn sonar:sonar"
					}
			}
       }
        stage('Upload artifact to Nexus') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'java-web-app', classifier: '', file: 'target/java-web-app-5.0.war', type: 'war']], credentialsId: 'nexuscreds', groupId: 'com.mt', nexusUrl: '54.219.213.138:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexus-hosted-repo', version: '5.0'
                }
               
        }
        
        stage("Build Docker Image") {
            steps{
               sh "docker build -t puneetgavri/javaapp:${BUILD_NUMBER} ."
            }
        }
        
        stage("Docker Push"){
            steps{
            sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            sh "docker push puneetgavri/javaapp:${BUILD_NUMBER}"
            }
        }
       
        stage("Deployment"){
            steps{
                sh 'sudo su && ssh -o StrictHostKeyChecking=no 52.53.212.90 docker run -td --name appcontainer1 -p 80:8080 puneetgavri/javaapp:${BUILD_NUMBER}'
            }
        }
    }
	post{
        always{
            echo "========always========"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}
