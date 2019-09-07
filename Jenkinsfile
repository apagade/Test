pipeline {
  agent any
  stages {
    stage('Stage1') {
      parallel {
        stage('Stage1') {
          steps {
            echo 'Step1'
            echo 'Build Assets'
          }
        }
        stage('Stage2') {
          steps {
            echo 'Step2'
            build 'ex-maven-project'
          }
        }
      }
    }
    stage('BlueStage1') {
      steps {
        sleep 1
      }
    }
  }
  environment {
    Stage1 = ''
  }
}