pipeline {
    agent any 

tools {
        // Referencing the names of the tools configured in Jenkins
        maven 'Maven'  // Use the exact name of the Maven installation
        jdk 'Java-17'  // Use the exact name of the JDK installation
    }
   

    stages {
        stage('Checkout') { // Missing step to fetch code from GitHub
            steps {
                git url: 'https://github.com/Raaz1630/Capstone_project.git', branch: 'main'
            }
        }
 
          stage('Stage-0 : Static Code Analysis Using SonarQube') { 
           steps {
                sh 'mvn clean verify sonar:sonar'
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

        stage('Quality Gate Check') { // Conditional check for SonarQube quality gate
            steps {
                script {
                    timeout(time: 1, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }

           stage('Stage-8 : Deploy an Artifact to Artifactory Manager i.e. Nexus/Jfrog') { 
            steps {
                sh 'mvn deploy'
            }
        }

        stage('Stage-9 : Deploy WAR to Tomcat') { 
            steps {
                script {
                    def warFile = findFiles(glob: 'target/*.war')[0].name
                    sh """
                    curl -u admin:redhat@123  -T target/${warFile} "http://http://54.91.182.176:8080/manager/text/deploy?path=/inventory-app&update=true"
                    """
                }
            }
        }

        stage('Stage-10 : Smoke Test') { 
            steps {
                sh 'curl --retry-delay 10 --retry 5 "http://http://54.91.182.176/:8080/cbapp"'
            }
        }

        stage('AWS CloudWatch Setup') { // Missing CloudWatch setup
            steps {
                sh '''
                # Install and configure AWS CloudWatch agent
                sudo yum install -y amazon-cloudwatch-agent
                sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config \
                -m ec2 -c ssm:CloudWatchConfig -s
                '''
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
