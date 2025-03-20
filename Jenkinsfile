pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }
      stage('Unit Test') {
            steps {
              sh "mvn test"
              echo "Unit Test Completed"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco(execPattern: 'target/jacoco.exec')
              }
            }
        }
        stage('Docker Build & Push') {
            steps {
              withDockerRegistry(credentialsId: 'docker', url: "") {
                sh "docker build -t gurugreen/spring-boot-app:$GIT_COMMIT ."
                sh "docker push gurugreen/spring-boot-app:$GIT_COMMIT"
              }
            }
        }
        stage('K8s Deploy'){
            steps {
              withKubeConfig([credentialsId: 'kubeconfig', serverUrl: '']) {
                sh "sed -i 's#replace#gurugreen/spring-boot-app:$GIT_COMMIT#g' k8s/deployment.yaml"
                sh "kubectl apply -f k8s/deployment.yaml"
              }
            }
        }
    }
}