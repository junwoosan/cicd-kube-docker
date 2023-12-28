pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "junwoolee/vproappdock"
        registrycredentials = "dockerhub"
    }

    stages{

   
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
                scannerHome = tool 'sonarscanner'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=junwoo-sonar-test \
                   -Dsonar.projectName=junwoo-sonar-test \
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

        stage('Build Docker App Image'){
            steps {
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }
        stage('Upload Image') {
            steps {
                script {
                    docker.withRegistry('',registrycredentials){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push("latest")
                        
                    }
                }
            }
        }
        stage('Remove unused docker image') {
            steps {
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }
        stage('Kubernetes deploy') {
            agent {label 'KOPS'}
            steps {
                sh 'helm upgrade --install --force --namespace prod vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER}'
            }
        }

        


    }


}