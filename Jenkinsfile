pipeline {
  agent none
  options {
    skipDefaultCheckout()
  }
  stages {
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
          image 'maven:3-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Build Backend'
        unstash 'ws'
        sh './mvnw -B -DskipTests=true clean compile package'
        stash name: 'war', includes: 'target/**/*.war'
      }
      post {
        success {
          archive 'target/**/*.war'
        }
      }
    }

    stage('Test Backend') {
      agent {
        docker {
          image 'maven:3-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Test backend'
        unstash 'ws'
        unstash 'war'
        sh './mvnw -B test findbugs:findbugs'
      }
    }

    stage('More Tests') {
      agent {
        docker {
          image 'maven:3-alpine'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'more tests'
        
      }
    }
  }
}
