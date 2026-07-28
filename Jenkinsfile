pipeline {
    agent none

    tools{
        maven 'mukeshmaven'
    }

    parameters {
        string(name: 'Env', defaultValue: 'Test', description: 'Version to deploy')
        booleanParam(name: 'executeTests', defaultValue: true, description: 'Decide to run tc')
        choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'])
    }

    environment {
        BUILD_SERVER = 'ec2-user@172.31.2.148'
        IMAGE_NAME = 'mukeshtho/addbook:$BUILD_NUMBER'
    }

    stages {
        stage('Compile') {
            agent any 
            steps {
                script{
                    // sshagent (['slave2']) {
                    echo 'package Hello World'
                    echo "compiling version ${params.APPVERSION}"
                    // sh scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER}:home/ec2-user"
                    // sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-script.sh'"
                    sh "mvn compile"
                    }
                }
            }
        }
        stage('UnitTest') {
            agent any
            when{
                expression { 
                    return params.executeTests == true
                }
            }
            steps {
                script{ 
                    echo 'Run UnitTest cases for Hello World'
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
            // agent { label 'linux_slave' }
            agent any
            steps {
                script{ 
                    echo 'CodeReview Hello World'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn pmd:pmd"
                }
            }
        }
        stage('CodeCoverage') {
            agent any
            steps {
                script{ 
                    echo 'Coverage Analysis Hello World'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn verify"
                }
            }
        }        
        // stage('Package') {
        //     agent any
        //     steps {
        //         script{ echo 'Package Hello World'
        //         echo "Packaging version ${params.APPVERSION}"
        //         sh "mvn package"

        //         }

     
        //     }
        // }
        // stage('PublishtoJfrog') {
        //     agent any
        //     when{
        //         expression { 
        //             return params.executeTests == true
        //         }
        //     }
        //     input {
        //         message 'archeive the artifact'
        //         ok 'platform selected'
        //         parameters {
        //             choice(name:'Platform',choices: ['Nexus', 'Jfrog'])
        //         }
        //     }
        //     steps {
        //         script{ 
        //             echo 'Publish to jfrog'
        //             echo "deploying in ${params.Env} environment"
        //             sh "mvn -U deploy -s settings.xml"
        //         }
        //     }
        // }
        stage('Dockerize the app and push the image') { build on server
            agent any
            steps{
                script{ 
                   sshagent (['slave2']) {

                    echo "containerizing the code and pushing image"
                    sh scp -o StrictHostKeyChecking=no server-script.sh ${BUILD_SERVER}:home/ec2-user"
                    sh "ssh -o StrictHostKeyChecking=no ${BUILD_SERVER} 'bash ~/server-script.sh {$IMAGE_NAME}'"
                    // sh "ssh -o strictHostKeyChecking=no ${BUILD_SERVER} 'docker build -t ${IMAGE_NAME} ."
                    sh "ssh ${BUILD_SERVER} sudo docker login -u abc -p xxxx"
                    sh "ssh ${BUILD_SERVER} sudo docker push ${IMAGE_NAME}"
                    //sh "ssh ${BUILD_SERVER} sudo docker run -itd -P ${IMAGE_NAME}"

                    }
                }
            }
        }
    }
}
