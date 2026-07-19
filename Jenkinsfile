pipeline {
    agent any

    tools{
        maven 'mukeshmaven'
    }

    parameters {
        string(name: 'Env', defaultValue: 'Test', description: 'Version to deploy')
        booleanParam(name: 'executeTests', defaultValue: true, description: 'Decide to run tc')
        choice(name: 'APPVERSION', choices: ['1.1', '1.2', '1.3'])
    }


    stages {
        stage('Compile') {
            steps {
                script{ 
                    echo 'Compile Hello World'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn compile"
                }
            }
        }
        stage('UnitTest') {
            when{
                expression{ 
                    params.executeTests == true
                }
            }
            steps {
                script{ echo 'Run UnitTest cases for Hello World'
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
            steps {
                script{ 
                    echo 'CodeReview Hello World'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn pmd:pmd"
                }
            }
        }
        stage('CodeCoverage') {
            steps {
                script{ 
                    echo 'Coverage Analysis Hello World'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn verify"
                }
            }
        }        
        stage('Package') {
            steps {
                script{ echo 'Package Hello World'
                echo "Packaging version ${params.APPVERSION}"
                sh "mvn package"

                }

     
            }
        }
        stage('PublishtoJfrog') {
            input {
                message 'archeive the artifact'
                ok 'platform selected'
                parameters {
                    choice(name:'Platform',choices: ['Nexus', 'Jfrog'])
                }
            }
            steps {
                script{ 
                    echo 'Publish to jfrog'
                    echo "deploying in ${params.Env} environment"
                    sh "mvn -U deploy -s settings.xml"
                }
            }
        }
    }
}
