pipeline{
    agent any
    tools{
        maven 'M3'
    }
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
                stash includes: 'target/hello-0.0.1.war,src/pt/Hello_World_Test_Plan.jmx',
                name: 'binary'
                
                    }
                

            }
        
        
                stage("performance testing"){
                    
                    agent {label 'docker-pt' }
                    stages{
                        stage('Start Tomcat'){
                            steps{
                                sh '''cd /home/jenkins/tomcat/bin
                                ./startup.sh''';
                            }
                        }
                        stage('deploy'){
                            steps{
                            unstash 'binary'
                            sh 'pwd'
                            sh 'ls target'
                            sh 'cp target/hello-0.0.1.war /home/jenkins/tomcat/webapps/';
                            }
}
                        stage("Performance Testing"){
                            steps{
                                sh '''cd /opt/jmeter/bin/
                                ./jmeter.sh -n -t $WORKSPACE/src/pt/Hello_World_Test_Plan.jmx -l $WORKSPACE/test_report.jtl''';
                                step([$class: 'ArtifactArchiver', artifacts: '**/*.jtl'])
                            }
                        }
                        stage("Promote build in artifactory"){
                            steps{
                            withCredentials([usernameColonPassword(credentialsId:
                                            'artifactory-account', variable: 'credentials')]) {
                            sh 'curl -u${credentials} -X PUT \
                            "http://192.168.123.152:8081/artifactory/api/storage/example-project/ \
                            ${BUILD_NUMBER}/hello-0.0.1.war?properties=PerformanceTested=Yes"';
                            }
                            }
                        }
                        }
                    }
                    
                }
    
    post{
        always{
            echo "========always========"
            echo "We made it! Yeah!"
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}