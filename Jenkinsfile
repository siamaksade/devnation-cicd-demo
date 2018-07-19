pipeline {
  agent {
    label 'maven'
  }
  stages {
    stage('Build') {
      steps {
        git url: 'https://github.com/spring-projects/spring-petclinic.git'
        sh "mvn package"
      }
    }
    stage('Deploy') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject() {
              def pom = readMavenPom file: 'pom.xml'
              openshift.startBuild("spring-petclinic", "--binary=true", "--from-file=spring-petclinic-${pom.version}.jar")
            }
          }
        }
      }
    }
  }
}

