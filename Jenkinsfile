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
}
}
