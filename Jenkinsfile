pipeline {
    agent any
    parameters {
        string(name: 'BRANCH_NAME', defaultValue: 'fe/dev', description: 'Specify the branch name to deploy')
    }
    stages {
        stage("GIT checkout") {
            steps {
                script {
                    // Checkout the specified branch
                    def branchName = params.BRANCH_NAME
                    checkout([$class: 'GitSCM', branches: [[name: branchName]], userRemoteConfigs: [[url: 'https://github.com/devopsdump/Groompet.git']]])
                }
            }
         }
        stage("Testing") {
            steps {
                echo "Test Runs"
            
            }
        }
        stage("Build") {
            steps {
                sh "mvn clean package"
                sh "mv target/Groompet-1.0.war target/Groompet.war"
            }
        }
        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'fe/dev' }
            }
            steps {
                deployToTomcat('43.205.238.180', 'tomcat', 'tomcat', 'http://43.205.238.180:8080/manager/text', '/Groompet.war, 'Dev')
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'fe/qa' }
            }
            steps {
                deployToTomcat('13.201.47.166', 'tomcat', 'tomcat', 'http://13.201.47.166:8080/manager/text', '/Groompet.war', 'QA')
            }
        }
        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'master' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
                deployToTomcat('65.2.187.108', 'tomcat', 'tomcat', 'http://65.2.187.108:8080/manager/text', '/Groompet.war', 'Prod')
                
                // Send Slack notification
                slackSend(channel: '8pm-devops-batch', message: "Deployment to PROD has been approved by manager.")
            }
        }
    }
}

def deployToTomcat(tomcatIP, username, password, tomcatURL, contextPath, environment) {
    def warFileName = 'target/Groompet.war'

    // Deploy the WAR file using curl
    sh """
        curl -v -u ${username}:${password} --upload-file ${warFileName} ${tomcatURL}/deploy?path=${contextPath}&update=true
    """
    echo "Deployment to ${environment} server completed."
}
