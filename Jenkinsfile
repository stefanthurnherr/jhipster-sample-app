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
      post {
        success {
          junit '**/surefire-reports/**/*.xml'
          findbugs pattern: 'target/**/findbugsXml.xml', unstableNewAll: '0' //unstableTotalAll: '0'
        }
        unstable {
          junit '**/surefire-reports/**/*.xml'
          findbugs pattern: 'target/**/findbugsXml.xml', unstableNewAll: '0' //unstableTotalAll: '0'
        }
      }
    }

    stage('More Tests') {
      agent none
      when {
        anyOf {
            branch "master"
            branch "release-*"
        }
      }
      steps {
        parallel(
        'Frontend' : {
            script {
                node {
                    unstash 'ws'
                    //sh 'gulp test'
                    sh './frontEndTests.sh'
                }
            }
        },
        'Performance' : {
            script {
                node {
                    docker.image('maven:3-alpine').inside('-v $HOME/.m2:/root/.m2') {
                        unstash 'ws'
                        unstash 'war'
                        sh './mvnw -B gatling:execute'
                    }
                }
            }
        })
      }
    }
  }
}
