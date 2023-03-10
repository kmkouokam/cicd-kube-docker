pipeline {

    agent any

	tools {
        maven "maven3"
        jdk "OracleJDK8"
    }

    environment {
         registry = "kmkouokam/vproappdock"
         registryCredential = "dockerhub"
    }

    stages{

       /* stage('Fetch Code') {
            steps {
                git branch: 'master', url: 'https://github.com/kmkouokam/cicd-kube-docker.git'
            }
          }
          */
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'sonar4.8'
            }

            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker App Image') {
            steps {
              script {
                 dockerImage = docker.build registry + ":V$BUILD_NUMBER"
              }
            }
        }

        stage('Upload Image to dockerhub') {
            steps {
               script {
                  docker.withRegistry( '', registryCredential) {
                  dockerImage.push("V$BUILD_NUMBER")
                  dockerImage.push("latest")
                  }
               }
            }
        }

        stage('Remove Unused Docker image') {
            steps {
              sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('K8s Deploy') {
           agent {label 'KOPS'}

              steps {
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --namespace prod"
              }
        }




    }


}