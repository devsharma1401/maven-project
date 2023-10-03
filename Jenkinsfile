pipeline {
           agent any
                   stages {
                           stage ("pull the code from scm") {
                                   agent {
                                           label "slave-1"
                                   }
                                   steps {
                                            git branch: 'main', url: 'https://github.com/devsharma1401/new-pipeline.git'
                                   }
                        }
                           stage ("build the code") {
                                   agent {
                                           label "slave-1"
                                   }
                                   steps {
                                           sh 'sudo mvn dependency:purge-local-repository'
                                           sh 'sudo mvn clean package'
                                   }
                           }
                           stage ("build the docker image") {
                                   agent {
                                           label "slave-1"
                                   }
                                   steps {
                                           sh 'sudo docker build -t new-java-app:$BUILD_TAG .'
                                           sh 'sudo docker tag new-java-app:$BUILD_TAG dev1401/new-java-app:$BUILD_TAG'
                                   }
                           }
                           stage ("push the docker-hub") {
                                   agent {
                                           label "slave-1"
                                   }
                                   steps {
                                            withCredentials([usernamePassword(credentialsId: 'docker_crede', passwordVariable: 'docker_cred_passwd', usernameVariable: 'docker_cred_user')]) {
                                                     sh 'sudo docker login -u ${docker_cred_user} -p ${docker_cred_passwd}'
                                                     sh 'sudo docker push dev1401/new-java-app:$BUILD_TAG'
}

                                   }
                           }
                           stage ("launch the docker-container") {
                                   agent {
                                          label "slave-3"
                                   }
                                   steps {
                                            
					    
					    sh 'sudo docker pull dev1401/new-java-app:$BUILD_TAG'
                                            
					    sh 'sudo docker run -dit --name dev -p 8080:8080 dev1401/new-java-app:$BUILD_TAG'
                                   }
                           }
                          stage ("testing") {
			           agent {
                                           label "slave-3"
				     }
			           steps {
				           retry (5){
					              script {
						               sh 'curl http://172.31.43.190:8080/java-web-app/ | grep -i india'
						      }
					   }
				   }
			  }
                           stage ("for approval") {
                                   steps {
                                           script {
                                          Boolean userInput = input(id: 'Proceed1', message: 'Do you want to Promote the build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '' , name: 'Please confirm you agree with this']])               				
                                          echo 'userInput: ' + userInput
                                           }
                                   }
                           }
			   stage ("prod env") {
			           steps {
				           sshagent(['production']) {
					   sh "ssh -o strictHostkeychecking=no ubuntu@172.31.9.9 sudo docker run -d -p 49513:8080 dev1401/new-java-app:$BUILD_TAG"
					   }

				   }
			   }

                   }
           
    }
