pipeline {
    agent any 
    
    tools{
        jdk 'jdk11'
        maven 'maven3'
    }
     parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'fe/dev', description: 'Specify the branch name to deploy')
    }
    stages{
        
        stage("Git Checkout"){
            steps{
                  def branchName = params.BRANCH_NAME
                git branches: [[name: branchName]], changelog: false, poll: false, url: 'https://github.com/jaiswaladi246/Petclinic.git'
            }
        }
        
        stage("Compile"){
            steps{
                sh "mvn clean compile"
            }
        }
        
         stage("Test Cases"){
            steps{
                sh "mvn test"
            }
        }
        
        
         stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
         stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'fe/dev' }
            }
            steps {
                sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'fe/QA' }
            }
            steps {
                sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
            }
        }
         stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'master' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
               sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/"
                
                // Send Slack notification
                slackSend(channel: '8pm-devops-batch', message: "Deployment to PROD has been approved by manager.")
            }
        }
}
