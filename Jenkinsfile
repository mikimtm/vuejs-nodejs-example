pipeline {
    agent {
        docker { image 'node:14-alpine' }
    }

  stages {
    stage('Docker Build') {
      steps {
        sh "docker build -t mikicloud/vuejs-node-example:${env.BUILD_NUMBER} ."
      }
    }
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
          sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
          sh "docker push mikicloud/vuejs-node-example:${env.BUILD_NUMBER}"
        }
      }
    }
    stage('Docker Remove Image') {
      steps {
        sh "docker rmi mikicloud/vuejs-node-example:${env.BUILD_NUMBER}"
      }
    }
    stage('Apply Kubernetes Files') {
      steps {
          withKubeConfig([credentialsId: 'kubeconfig']) {
          sh 'cat deployment.yaml | sed "s/{{BUILD_NUMBER}}/$BUILD_NUMBER/g" | kubectl apply -f -'
          sh 'kubectl apply -f service.yaml'
        }
      }
  }
}
post {
    success {
      slackSend(message: "Pipeline is successfully completed.")
    }
    failure {
      slackSend(message: "Pipeline failed. Please check the logs.")
    }
}
}
