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
                sh "mv target/petclinic-1.0.war target/petclinic.war"
            }
        }
        stage("Deploy to Dev") {
            when {
                expression { params.BRANCH_NAME == 'fe/dev' }
            }
            steps {
                deployToTomcat('34.233.120.114', 'tomcat', 'tomcat', 'http://34.233.120.114:8080/manager', '/petclinic.war', 'Dev')
            }
        }
        stage("Deploy to QA") {
            when {
                expression { params.BRANCH_NAME == 'fe/qa' }
            }
            steps {
                deployToTomcat('3.90.109.218', 'tomcat', 'tomcat', 'http://3.90.109.218:8080/manager', '/petclinic.war', 'QA')
            }
        }
        stage("Deploy to Prod") {
            when {
                expression { params.BRANCH_NAME == 'main' }
            }
            steps {
                input(message: "Do you want to proceed to PROD?", ok: "Proceed") // Approval step

                // Deploy to PROD server after approval
                deployToTomcat('54.90.253.25', 'tomcat', 'tomcat', 'http://54.90.253.25:8080/manager', '/petclinic.war', 'Prod')
                
                // Send Slack notification
                slackSend(channel: '8pm-devops-batch', message: "Deployment to PROD has been approved by manager.")
            }
        }
    }
}

def deployToTomcat(tomcatIP, username, password, tomcatURL, contextPath, environment) {
    def warFileName = 'target/petclinic.war'

    // Deploy the WAR file using curl
    sh """
        curl -v -u ${username}:${password} --upload-file ${warFileName} ${tomcatURL}/deploy?path=${contextPath}&update=true
    """
    echo "Deployment to ${environment} server completed."
}
