// if ("${env.GIT_BRANCH}" == 'origin/main') {
// 	    env.instanceIP="35.205.107.69"
// } else if ("${env.GIT_BRANCH}" == 'origin/development') {
// 	    env.instanceIP="34.140.196.201"
// }

pipeline {
	agent any
	environment { // GIVE THESE VALUES
		//appIP="$instanceIP";
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
			script {
					if ("${GIT_BRANCH}" == 'origin/main') {
						sh '''
						ssh -i "~/.ssh/id_rsa" jenkins@35.205.107.69 << EOF
						docker rm -f javabuild
						'''
					} else if ("${GIT_BRANCH}" == 'origin/development') {
						sh '''
						ssh -i "~/.ssh/id_rsa" jenkins@34.140.196.201 << EOF
						docker rm -f javabuild
						'''
					}
					}
                }
		stage('Restart app'){
			steps{
			script {
					if ("${GIT_BRANCH}" == 'origin/main') {
						sh '''
						ssh -i "~/.ssh/id_rsa" jenkins@35.205.107.69 << EOF
						docker run -d -p 8080:8080 --name $containerName $imageName:latest
						'''
					} else if ("${GIT_BRANCH}" == 'origin/development') {
						sh '''
						ssh -i "~/.ssh/id_rsa" jenkins@34.140.196.201 << EOF
						docker run -d -p 8080:8080 --name $containerName $imageName:latest
						'''
					}
				}
			}
		}
	}
}
