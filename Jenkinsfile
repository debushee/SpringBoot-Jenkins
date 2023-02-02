pipeline {
	agent any
	environment { // GIVE THESE VALUES
		appIP="34.77.153.124";
		containerName="java-docker";
		imageName="java-docker";
	}
	stages{
		stage('Docker Build'){
			steps{
			sh '''
			docker build -t debushee/$imageName:latest -t debushee/$imageName:build-$BUILD_NUMBER .
			'''
			}
		}
		stage('Push Images'){
			steps{
			sh '''
			docker push debushee/$imageName:latest
			docker push debushee/$imageName:build-$BUILD_NUMBER
			'''
			}
                }
		stage('Restart Container'){
			steps{
			sh '''
			ssh -i "~/.ssh/jenkins_key" jenkins@$appIP << EOF
				if [ ! "$(docker ps -a -q -f name=$containerName)" ]; then
    				if [ "$(docker ps -aq -f status=exited -f name=$containerName)" ]; then
        				docker rm -f $containerName
						docker rmi $imageName
    				fi
    				# run your container
    				docker run -d -p 8080:8080 --name $containerName $imageName
				fi
			'''
			}
		}
		stage('Clean Up'){
			steps{
			sh '''
			docker system prune -f
			'''
			}
		}
	}
}
