pipeline {
   agent any

       parameters {
           booleanParam(defaultValue: false, description: 'Do you want to deploy the application?', name: 'deployment')
       }

       environment {
           ARTIFACT_ID = readMavenPom().getArtifactId()
           ARTIFACT_VERSION = readMavenPom().getVersion()
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
    }
}
