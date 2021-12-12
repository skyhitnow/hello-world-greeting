pipeline{
    agent none
    tools{
        maven 'M3'
    }
    stages{

 stage("whole"){
            agent any
            stages{
stage("SCM"){
            steps{
                git 'https://github.com/skyhitnow/hello-world-greeting'
            }
            
        }
        stage('Build & Unit Tests'){
            steps{
                sh 'mvn clean verify -DskipITs=true'
                junit '**/target/surefire-reports/TEST-*.xml'
                archive 'target/*.jar'
            }
        }

        stage("static code analysis"){
            steps{
                withSonarQubeEnv('local-sonarqube-server'){
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project \
                    -Dsonar.projectKey=example-project \
                    -Dsonar.projectVersion=$BUILD_NUMBER'
                println "${env.SONAR_HOST_URL}"
                }


                //sh 'mvn clean verify sonar:sonar -Dsonar.projectName=example-project \
               // -Dsonar.projectKey=example-project \
               // -Dsonar.projectVersion=$BUILD_NUMBER'
            }
        }

        stage ('Integration Test'){
            steps{
                sh 'mvn clean verify -Dsurefire.skip=true'
                junit '**/target/failsafe-reports/TEST-*.xml'
                archive 'target/*.jar'
                }
            
}
        stage ('Publish'){
                    steps{
                        //script{
                            //def server = Artifactory.server 'local-artifactory-server'
               // def uploadSpec = """{
               // "files": [
                //{
               // "pattern": "target/hello-0.0.1.war",
               // "target": "helloworld-greeting-project/${BUILD_NUMBER}/",
               // "props": "Integration-Tested=Yes;Performance-Tested=No"

               rtUpload (
                    // Obtain an Artifactory server instance, defined in Jenkins --> Manage Jenkins --> Configure System:
                    serverId: 'local-artifactory-server',
                    spec: """{
                            "files": [
                                    {
                                        "pattern": "target/hello-0.0.1.war",
                                        "target": "example-project/${BUILD_NUMBER}/"
                                    }
                                ]
                            }"""
                )
                
                    }
                
}
            }
        }
        
                stage("stash"){
                    steps{echo "NODE_NAME = ${env.NODE_NAME}"}
                    agent {
                        docker {
                            label 'docker-pt'  
                            image ' performance-tester-agent-0.1'  
                            reuseNode true
                        }     
                    }
                    steps{
                        echo "Hellow world"
                    }
                }
    }
    post{
        always{
            echo "========always========"
            echo "this is a always part"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}