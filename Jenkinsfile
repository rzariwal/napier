pipeline {
   agent any

    parameters {
        // booleanParam, choice, file, text, password, run, or string
        booleanParam(defaultValue: false, description: 'Do you want to deploy the application?', name: 'deployment')
        //string(defaultValue: "TEST", description: 'What environment?', name: 'stringExample')
        //text(defaultValue: "This is a multiline\n text", description: "Multiline Text", name: "textExample")
        //choice(choices: 'US-EAST-1\nUS-WEST-2', description: 'What AWS region?', name: 'choiceExample')
        //choice(choices: ['greeting' , 'silence'],description: '', name: 'REQUESTED_ACTION')
        //password(defaultValue: "Password", description: "Password Parameter", name: "passwordExample")
    }

   environment {
       //Use Pipeline Utility Steps plugin to read information from pom.xml into environment variables
       ARTIFACT_ID = "latest"
       ARTIFACT_VERSION = "1"
       DEPLOYMENT= "${params.deployment}"
       REGISTRY = "zariwal/capstone"
       SERVICE_NAME = "spring-boot-kubernetes"
       REPOSITORY_TAG = "${YOUR_DOCKERHUB_USERNAME}/${ORGANIZATION_NAME}-${SERVICE_NAME}:${BUILD_ID}"
       DOCKERHUB = 'dockerhub_credentials'
       PROJECT_NAME = 'spring-boot-kubernetes'
   }

    stages {
        stage ('OWASP Dependency-Check Vulnerabilities') {
              steps {
                  dependencyCheck additionalArguments: '''
                        -o "./"
                        -s "./"
                        -f "ALL"
                        --disableAssembly
                        --disableNodeJS
                        --disableYarnAudit
                        --prettyPrint''', odcInstallation: 'Default'

                  dependencyCheckPublisher pattern: 'dependency-check-report.xml'
              }
        }

        stage('Sonarqube Analysis') {
              environment {
                  scannerHome = tool 'SonarQube'
            }
            steps {
                  withSonarQubeEnv('SonarQube') {
                      sh "${scannerHome}/bin/sonar-scanner -Dsonar.exclusions=src/**/*,dependency-check-report*.* -Dsonar.userHome=`pwd`/.sonar -Dsonar.projectKey=$PROJECT_NAME -X"
                  }
            }
        }

        stage ('Build Application') {
            steps {
                sh 'mvn clean install'
            }
            post {
                success {
                    junit 'target/surefire-reports/**/*.xml'
                }
                failure {
                    sh 'echo Build failed, Sending notification....'
                }
            }
        }

        stage('Building our docker image') {
            steps {
                script {
                    dockerImage = docker.build(registry +":$BUILD_NUMBER", "-f deploy/Dockerfile .")
                }
            }
        }

        stage('Container Image Scan') {
           steps {
                sh 'echo "zariwal/capstone:61 `pwd`/deploy/Dockerfile" > anchore_images'
                anchore name: 'anchore_images'
           }
        }

        stage('Push our image dockerhub') {
            steps {
                script {
                    docker.withRegistry( '', DOCKERHUB ) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deploy to kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes_credentials',
                        serverUrl: 'https://192.168.100.43:6443',
                        namespace: 'napier'
                        ]) {
                        sh 'envsubst < deploy/kubernetes.yml | kubectl apply -f -'
                }
            }
        }

    	stage ("Dynamic Analysis - DAST with OWASP ZAP") {
			steps {
				sh "docker run --network host -t owasp/zap2docker-stable zap-baseline.py -t http://192.168.100.43:31000 || true"
			}
	    }

    }
}
