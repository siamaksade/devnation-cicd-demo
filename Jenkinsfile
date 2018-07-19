pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build') {
      steps {
        sh "cp .settings.xml ~/.m2/settings.xml"
        git url: 'https://github.com/siamaksade/mapit-spring.git'
        sh "mvn package -DskipTests"
      }
    }
    stage('Test') {
      steps {
        sh "mvn test"
      }
    }
  }
}