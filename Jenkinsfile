pipeline {
  agent any
  environment { 
    docker_username = 'harilaleficode'
  }
  stages {
    stage('Clone down') {
      steps {
        stash(excludes: '.git', name: 'code')
      }
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          options {
            skipDefaultCheckout()
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            stash 'code'
            archiveArtifacts 'app/build/libs/'
            sh 'ls'
            deleteDir()
          }
        }
      }
    }

    stage('push docker app') {
      options {
        skipDefaultCheckout()
      }
      when {
        branch 'master'
      }
      environment {
        DOCKERCREDS = credentials('docker_login')  //use the credentials just created in this stage
      }
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
      }
    }

    stage('component test') {
      when { 
        expression {
            return env.BRANCH_NAME != 'dev/*';
        } 
      }
      steps {
        sh 'ci/component-test.sh'
      }
    }
    
  }
}
