pipeline {
  agent any
    stage('Checkout') {
      agent any
      steps {
        echo 'Checkout'
        checkout scm
        stash(name: 'ws', includes: '**')
      }
    }
    stage('Build Backend') {
      agent {
        docker {
          image: 'maven:3-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Build Backend'
        unstash 'ws'
      }
      post {
        success {
          archive 'target/**/*.war'
        }
      }
    }

    stage('More Tests') {
      steps {
        echo 'more tests'
      }
    }
  }
}
