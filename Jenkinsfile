if ("${GIT_BRANCH}" == 'origin/main') {
	    appIP="35.205.107.69"
} else if ("${GIT_BRANCH}" == 'origin/development') {
	    appIP="34.140.196.201"
}

pipeline {
	agent any
	environment { // GIVE THESE VALUES
		//appIP="146.148.9.120";
		containerName="java-docker";
		imageName="debushee/java-docker";
	}
	stages{
		stage('Test application'){
			steps{
			sh '''
			mvn clean test
			'''
			}
		}
		stage('Save tests'){
			steps{
			sh 'mkdir -p /home/jenkins/Tests/${BUILD_NUMBER}_tests/'
			sh 'mv ./target/surefire-reports/*.txt /home/jenkins/Tests/${BUILD_NUMBER}_tests/'
			}
		}
		stage('War Build'){
			steps{
			sh '''
			mvn clean package
			'''
			}
		}
		stage('Moving war'){
			steps{
			sh '''
			mkdir -p ./wars
			mv ./target/*.war ./wars/project_war.war	
			'''
			}
		}
		stage('Docker Build'){
			steps{
			sh '''
			docker build -t $imageName:latest -t $imageName:build-$BUILD_NUMBER .
			'''
			}
		}
		stage('Push Images'){
			steps{
			sh '''
			docker push $imageName:latest
			docker push $imageName:build-$BUILD_NUMBER
			'''
			}
                }
		stage('Stopping container'){
			steps{
			sh '''
			ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			docker rm -f $containerName	
			'''
			}
                }
		stage('Restart app'){
			steps{
			sh '''
			ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
			docker run -d -p 8080:8080 --name $containerName $imageName
			'''
			}
		}
	}
}
