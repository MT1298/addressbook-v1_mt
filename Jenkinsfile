pipeline {
    agent none

    tools {
        maven 'mukeshmaven'
    }

    parameters {
        string(name: 'Env', defaultValue: 'Test', description: 'Version to deploy')
        booleanParam(name: 'executeTests', defaultValue: true, description: 'Decide to run tc')
        choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'], description: 'Application Version')
    }

    environment {
        BUILD_SERVER = 'ec2-user@172.31.5.165'
        IMAGE_NAME = "mukeshtho/addbook:${BUILD_NUMBER}"
    }

    stages {

        stage('Compile') {
            agent any
            steps {
                script {
                    echo 'Compiling Hello World'
                    echo "Compiling version ${params.APPVERSION}"
                    sh 'mvn compile'
                }
            }
        }

        stage('UnitTest') {
            agent any

            when {
                expression {
                    return params.executeTests
                }
            }

            steps {
                script {
                    echo 'Running Unit Tests'
                    sh 'mvn test'
                }
            }

            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('CodeReview') {
            agent any

            steps {
                script {
                    echo 'Code Review'
                    echo "Deploying in ${params.Env} environment"
                    sh 'mvn pmd:pmd'
                }
            }
        }

        stage('CodeCoverage') {
            agent any

            steps {
                script {
                    echo 'Coverage Analysis'
                    echo "Deploying in ${params.Env} environment"
                    sh 'mvn verify'
                }
            }
        }

        stage('Dockerize the app and push the image') {

            agent any

            steps {

                script {

                    sshagent(['slave2']) {
					
                        withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        echo "Containerizing the application"

                        sh "scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER}:/home/ec2-user/"

                        sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-script.sh ${IMAGE_NAME}'"

                        sh "ssh ${BUILD_SERVER} 'sudo docker login -u ${USERNAME} -p ${PASSWORD}'"

                        sh "ssh ${BUILD_SERVER} 'sudo docker push ${IMAGE_NAME}'"

                        // sh "ssh ${BUILD_SERVER} 'sudo docker run -itd -P ${IMAGE_NAME}'"
						
					    }	

                    }

                }

            }

        }

    }
}