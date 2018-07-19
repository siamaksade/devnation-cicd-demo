pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build') {
      steps {
        git url: 'https://github.com/spring-projects/spring-petclinic.git'
        sh "mvn package -DskipTests"
      }
    }
    stage('Build') {
      steps {
        sh "mvn test"
      }
    }
  }
}