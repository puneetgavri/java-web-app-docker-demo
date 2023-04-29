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
      // stage('SonarQube analysis') {
        //    steps{
          //        withSonarQubeEnv('mysonarserver') { 
            //      sh "mvn sonar:sonar"
		//			}
		//	}
       //}
        stage('Upload artifact to Nexus') {
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'java-web-app', classifier: '', file: 'target/java-web-app-5.0.war', type: 'war']], credentialsId: 'nexus_creds', groupId: 'com.mt', nexusUrl: '54.163.107.177:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'nexus-hosted-repo', version: '5.0'
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
       
       stage('downloadkey') {
              steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "aws_creds",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                 sh 'aws s3 cp s3://aaanewtestbucket/testkpNV.pem .'
                }
            }
        }
	 stage('create ec2') {
             steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "aws_creds",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                 sh 'aws ec2 run-instances --image-id ami-03c7d01cf4dedc891 --instance-type t2.medium --security-group-ids sg-0c753e8d4586a889f --key-name testkpNV --tag-specifications \'ResourceType=instance,Tags=[{Key=Name,Value=Demoec2}]\' --region us-east-1'
                script {
		def myinstanceid = sh(script: "aws ec2 describe-instances --filters 'Name=tag:Name,Values=Demoec2' --output text --query 'Reservations[*].Instances[*].InstanceId' --region us-east-1", returnStdout: true).trim() 
				println "instance id of the instance is ${myinstanceid}"
				env.INSTANCE_ID = myinstanceid
			}
		
		}
            }
        }
	stage('Get IP') {
             steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: "aws_creds",
                    accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                    secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]) {
                script {
		def myIP = sh(script: "aws ec2 describe-instances --instance-ids ${INSTANCE_ID} --query 'Reservations[].Instances[].PublicIpAddress' --output text --region us-east-1", returnStdout: true).trim() 
				println "IP of the instance is ${myIP}"
				env.IP = myIP
			}
		
		}
            }
        }
	    
	  stage("ssh to ec2") {
                steps {
                    script {
			    sh "chmod 400 NVkeypair.pem && ssh -o StrictHostKeyChecking=no -i testkpNV.pem ec2-user@${env.IP} 'sudo yum update -y && sudo yum install docker -y && sudo systemctl start docker && sudo usermod -a -G docker ec2-user && sudo chmod 755 /var/run/docker.sock && sudo docker pull fabinta/myjenkins_project:${BUILD_NUMBER} && sudo docker run -td --name mydemocontainer fabinta/myjenkins_project:${BUILD_NUMBER}'"
                    }
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
