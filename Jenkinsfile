pipeline {
    agent any 

    tools {
        maven 'Maven'
        jdk 'Java-17'
    }
  
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Raaz1630/Capstone_project.git', branch: 'main'
            }
        }

        stage('Static Code Analysis Using SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube-10.6') {
                    sh 'mvn clean verify sonar:sonar'
                }      
            }
        }

        stage('Stage-1 : Clean') { 
            steps {
                sh 'mvn clean'
            }
        }

        stage('Stage-2 : Validate') { 
            steps {
                sh 'mvn validate'
            }
        }

        stage('Stage-3 : Compile') { 
            steps {
                sh 'mvn compile'
            }
        }

        stage('Stage-4 : Test') { 
            steps {
                sh 'mvn test'
            }
        }

        stage('Stage-5 : Install') { 
            steps {
                sh 'mvn install'
            }
        }

        stage('Stage-6 : Verify') { 
            steps {
                sh 'mvn verify'
            }
        }

        stage('Stage-7 : Package') { 
            steps {
                sh 'mvn package'
            }
        }

        stage('Quality Gate Check') { 
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('Deploy to Artifactory') { 
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Deploy WAR to Tomcat') { 
            steps {
                script {
                    def warFiles = findFiles(glob: 'target/*.war')
                    if (warFiles.length > 0) {
                        def warFile = warFiles[0].name
                        sh """
                        curl -u admin:redhat@123 -T target/${warFile} "http://44.204.149.52:8080/manager/text/deploy?path=/inventory-app&update=true"
                        """
                    } else {
                        error "WAR file not found in target directory!"
                    }
                }
            }
        }

        stage('Smoke Test') { 
            steps {
                sh 'curl --retry-delay 10 --retry 5 "http://44.204.149.52:8080/inventory-app"'
            }
        }

        stage('AWS CloudWatch Setup') { 
            steps {
                script {
                    // Check if CloudWatch agent is running
                    def agentRunning = sh(script: 'sudo systemctl status amazon-cloudwatch-agent', returnStatus: true) == 0
                    if (!agentRunning) {
                        error "AWS CloudWatch agent is not running. Please check the agent configuration."
                    }
                    
                    // Verify configuration
                    def configFile = '/opt/aws/amazon-cloudwatch-agent/etc/amazon-cloudwatch-agent.json'
                    def configExists = sh(script: 'test -f ${configFile}', returnStatus: true) == 0
                    if (!configExists) {
                        error "CloudWatch configuration file not found at ${configFile}. Please check the configuration."
                    }
                    
                    echo "AWS CloudWatch Agent setup verified successfully."
                }
            }
        }
    }


    post {
        always {
            echo 'Cleaning up workspace'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
