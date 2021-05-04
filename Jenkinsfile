def sonarScanner(projectKey) {
    def scannerHome = tool 'sonarqube-scanner'
    withSonarQubeEnv("sonarqube") {
        if(fileExists("sonar-project.properties")) {
            sh "${scannerHome}/bin/sonar-scanner"
        }
        else {
            sh "${scannerHome}/bin/sonar-scanner -     Dsonar.projectKey=${projectKey} -Dsonar.java.binaries=build/classes -Dsonar.java.libraries=**/*.jar -Dsonar.projectVersion=${BUILD_NUMBER}"
        }
    }
    timeout(time: 10, unit: 'MINUTES') {
        waitForQualityGate abortPipeline: true
    }
}
pipeline {
  agent any
  stages {
    stage('Gradle Build') {
      steps {
        sh './gradlew clean build'
        script {
          docker.build("pipeline-as-code-app:${env.BUILD_ID}")
        }
      }
    }
    stage('SonarQube code scan') {
      steps {
        script {
            sonarScanner('category-service')
        }
      }
    } 
    stage('Tests') {
      parallel {
        stage('UT') {
          agent {
            docker {
              image 'openjdk:11-stretch'
            }
          }
          steps {
            sh './gradlew test'
          }
        }

        stage('IT') {
          steps {
            sh './gradlew integrationTest'
          }
        }
      }
    }
  }
  triggers {
    pollSCM('')
  }
}
