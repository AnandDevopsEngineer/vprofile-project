pipeline {
	agent any

	stages {
		
		stage('Checkout') {
			steps {
				git branch: 'docker',
				url: 'https://github.com/AnandDevopsEngineer/vprofile-project.git'
			}
		}

		stage('Build and Test') {
			steps {
				sh 'mvn clean package'
			}
		}
		
		stage('Docker Build'){
			steps {
				sh 'docker build -t vprofile-app:jenkins-${BUILD_NUMBER} -f Docker-files/wrong/Dockerfile .'
			}
		}

		stage('Docker Push'){
			steps {
				withCredentials([usernamePassword(
					credentialsId: 'dockerhub-creds',
					usernameVariable: 'DOCKER_USER',
					passwordVariable: 'DOCKER_PASS'
				)]) {
					sh ''' 
						echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
						
						docker tag \
							vprofile-app:jenkins-${BUILD_NUMBER} \
							ananddevopsengineer/vprofile-app:jenkins-${BUILD_NUMBER}

						docker push \
							ananddevopsengineer/vprofile-app:jenkins-${BUILD_NUMBER}
					    ''' 
				    }
				}

		}

		stage('Deploy') {
			steps {
				sh ''' 
					docker rm -f vprofile-app || true
					
					docker run -d \
					--name vprofile-app \
					--network compose_default \
					-p 8081:8080 \
					ananddevopsengineer/vprofile-app:jenkins-${BUILD_NUMBER}
				'''
			}
		}

		stage('Verify'){
			steps {
				sh '''
            				sleep 20

            				STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8081)

            				echo "Application HTTP status: $STATUS"

            				if [ "$STATUS" = "200" ] || [ "$STATUS" = "302" ]; then
                				echo "Application verification successful"
            				else
                				echo "Application verification failed"
                				exit 1
            				fi
        			   '''
    			}
		}
   }

	post {
		success {
        		echo 'Pipeline completed successfully.'
    		}

    		failure {
        		echo 'Pipeline failed. Check the console output.'
    		}

    		always {
        		echo 'Pipeline execution finished.'
    		}
	}
 }
