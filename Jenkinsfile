pipeline{
    agent any
    environment{
        version = "${env.BUILD_ID}"
    }
    stages{
        stage("solar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }                   
                }
            }         
        }
        stage("Docker build and push image"){
            script{
                withCredentials([string(credentialsId: 'docker-pass', variable: 'docker-pass')]) {
                       sh '''
                         docker build -t manishmaurya/springapp:${version} .
                         docker login -u manishmaurya -p ${docker-pass}
                         docker push manishmaurya/springapp:${version}
                         docker rmi manishmaurya/springapp:${version}
                      '''
                }
            }
        }
    }
}