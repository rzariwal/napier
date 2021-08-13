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
       ARTIFACT_ID = "spring-boot-kubernetes-4.1.0"
       ARTIFACT_VERSION = ""
       DEPLOYMENT= "${params.deployment}"
   }

    stages {
        stage('List pods') {
            steps {
                withKubeConfig([credentialsId: 'kubernetes_credentials',
                        serverUrl: 'https://192.168.100.43:6443',
                        namespace: 'napier'
                        ]) {
                        sh 'kubectl get all'
                }
            }
        }

        stage ('OWASP Dependency-Check Vulnerabilities') {
              steps {
                  dependencyCheck additionalArguments: '''
                        -o "./"
                        -s "./"
                        -f "ALL"
                        --prettyPrint''', odcInstallation: 'Default'

                  dependencyCheckPublisher pattern: 'dependency-check-report.xml'
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
        stage ('Build Docker Image') {
            steps {
                sh """
                echo IMAGE: ${ARTIFACT_ID}
                echo VERSION: ${ARTIFACT_VERSION}
                docker build -f deploy/Dockerfile \
                --build-arg JAR_FILE=target/${ARTIFACT_ID}-${ARTIFACT_VERSION}.jar \
                --build-arg IMAGE_VERSION=${ARTIFACT_VERSION} \
                -t ${ARTIFACT_ID}:${ARTIFACT_VERSION} .
                """
            }
        }
    }
}
