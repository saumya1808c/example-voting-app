pipeline {
    agent none

      stages {
        stage('worker-build') {
        agent {
          docker{
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
                }
             }
          when{
                changeset "**/worker/**"
               }
            steps {
                echo 'this is worker build job'
                dir('worker'){
                sh 'mvn compile'
                }
            }
       }
       stage('worker-test') {
       agent {
         docker{
               image 'maven:3.6.1-jdk-8-alpine'
               args '-v $HOME/.m2:/root/.m2'
               }
            }
       when {
                changeset "**/worker/**"
            }
           steps {
               echo 'this is worker test job'
               dir('worker'){
               sh 'mvn clean test'
               }
           }
           }


        stage('worker-package'){
        agent {
          docker{
                image 'maven:3.6.1-jdk-8-alpine'
                args '-v $HOME/.m2:/root/.m2'
                }
             }
           when {
                    branch 'master'
                    changeset "**/worker/**"
                }
          steps{
              echo 'packaging worker app into a jarfile'
              dir('worker'){
                   sh 'mvn package -DskipTests'
                   archiveArtifacts artifacts: '**/target/*.jar', fingerprint:true
                }
          }
        }

        stage('worker-docker-package'){
          agent any
           when {
                     branch 'master'
                    changeset "**/worker/**"
                }
                steps{
                     echo 'Packging worker app with docker'
                  script{
                  docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                  def workerImage = docker.build("saumya043/worker:v${env.BUILD_ID}", "./worker")
                  workerImage.push()
                  workerImage.push("${env.BRANCH_NAME}")
               }
             }
          }
        }

        stage('result-build') {
        agent {
          docker{
           image 'node:8.16-alpine'
                }
              }
          when{
                changeset "**/result/**"
               }
            steps {
                echo 'this is result build job'
                dir('result'){
                sh 'npm install'
                }
            }
       }
       stage('result-test') {
       agent {
         docker{
          image 'node:8.16-alpine'
               }
             }
       when {
                changeset "**/result/**"
            }
           steps {
               echo 'this is result test job'
               dir('result'){
               sh 'npm install'
               sh 'npm test'
               }
           }
           }
           stage('result-docker-package'){
             agent any
              when {
                      branch 'master'
                       changeset "**/result/**"
                   }
                   steps{
                        echo 'Packging result app with docker'
                     script{
                     docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                     def resultImage = docker.build("saumya043/result:v${env.BUILD_ID}", "./result")
                     resultImage.push()
                     resultImage.push("${env.BRANCH_NAME}")
                  }
                }
             }
           }

           stage('vote-build') {
           agent {
             docker{
              image 'python:2.7.16-slim'
              args '--user root'
                   }
                 }
             when{
                   changeset "**/vote/**"
                  }
               steps {
                   echo 'this is vote build job'
                   dir('vote'){
                   sh 'pip install -r requirements.txt'
                   }
               }
           }
           stage('vote-test') {
           agent {
            docker{
             image 'python:2.7.16-slim'
             args '--user root'
                  }
                }
           when {
                   changeset "**/vote/**"
               }
              steps {
                  echo 'this is vote test job'
                  dir('vote'){
                  sh 'pip install -r requirements.txt'
                  sh 'nosetests  -v'
                  }
              }
              }
              stage('vote-docker-package'){
                agent any
                 when {
                          branch 'master'
                          changeset "**/vote/**"
                      }
                      steps{
                           echo 'Packging vote app with docker'
                        script{
                        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
                        def voteImage = docker.build("saumya043/vote:v${env.BUILD_ID}", "./vote")
                        voteImage.push()
                        voteImage.push("${env.BRANCH_NAME}")

                     }
                   }
                }
              }
}

    post{
    always{
    echo 'pipeline for worker is complete'
          }
    failure{
    slackSend (channel: "instavote-cd" , message:"Build failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
          }
    success{
    slackSend (channel: "instavote-cd" , message:"Build succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
    }
}