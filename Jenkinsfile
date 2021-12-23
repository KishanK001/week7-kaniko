pipeline {
  agent none
  stages {
    stage('example build') {
      agent { docker 'maven:3.8.1-adoptopenjdk-11' }
      steps {
        echo 'Hello Maven'
        sh 'mvn --version'
      }
    stage('Example test') {
      agent { docker 'openjdk:8-jre' }
      steps {
        echo 'Hello, JDK'
        sh 'java -version'
      }
    }
  }
}
 
