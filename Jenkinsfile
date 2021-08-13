pipeline {
   agent any

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
}
