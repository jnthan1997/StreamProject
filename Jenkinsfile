
pipeline{ //All pipeline scripts start here
    agent any // this define on what agent or where the build will be made
    tools{ // tools need to build the project it was mentioned on the Readme file on how to configure it. 
            // make sure you have the same name as you configure the tools on "Manage System" > "Tools"
        jdk 'jdk17' //name of the JDK on tools is jdk17
        nodejs 'node16'
    }
    environment { //Environment variables are key-value pairs that can be used to store configuration settings, paths, or other information.
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages { // This block where automation start
        stage('clean workspace'){ // this stage is for cleaning environment workspace
            steps{ // steps must be within the stage block. Steps are building procedure for different stage.
                cleanWs()
            }
            post{   //post blocks are for email and other posting purposes
                always{ /* this post step are within the block of Stage: Clean Workspace where a Email will be sent to your email
                        that we've configure*/
                    mail to: 'youremail@here.com',
                    subject: "New Build has started - ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                    body: "Build is on going. Check ${env.BUILD_URL}"
                    /* ${env.JOB_NAME} - this variable pull the jobname
                     ${env.BUILD_NUMBER} - Build number
                    ${env.BUILD_NUMBER} and pulls the URL of the jenkins link*/
                }
            }
        }
        stage('Checkout from Git'){ //pulls source code in the Github
            steps{
                git branch: 'main', url: 'https://github.com/SemiColon1214/StreamProject.git'
            }
        }
        stage("Sonarqube Analysis "){ 
            steps{
                withSonarQubeEnv('sonar-server') { // project name and project are the same depend on what projectname you have created in SonarQube
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps { //This Stage will pull the Quality gate result on SonarQube
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            }
            post{
                success{ //This post section has success and failure blocks so the developer can be notify regarding on the build
                    mail to: 'recipient@example.com',
                    subject: "Quality Gate Passed! - ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                    body: "The build has passed the Quality Gate ${env.BUILD_URL}"
                }
                failure{
                    script{
                         def consoleOutput = currentBuild.rawBuild.getLog(100) // Get last 100 lines of console log
                        mail to: 'recipient@example.com',
                        subject: "Jenkins Build Failed - ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                        body: """
                        Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.

                        Check console output at ${env.BUILD_URL} to view the full results.

                        Last 100 lines of console output:

                        ${consoleOutput}
                     """
                    }
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps { 
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{ // Building and pushing docker images to docker hub credentials are made to secret
                script{
                    withCredentials([string(credentialsId: 'TMDB-API', variable: 'TMDB_V3_API')]){
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=$TMDB_V3_API -t netflix ."
                       sh "docker tag netflix devopsnthn/netflix:latest "
                       sh "docker push devopsnthn/netflix:latest "
                    }
                }
            }
        }
        }
        stage("TRIVY"){
            steps{ //scanning image
                sh "trivy image devopsnthn/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{ // this will deploy your image to local computer
                sh 'docker run -d -p 8081:80 devopsnthn/netflix:latest'
            }
            post{
                success{
                    mail to: 'recipient@example.com',
                    subject: "Build Deployed Successfully! - ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                    body: "You may now view your application in browser IPAddress:8081"
                }
                failure{
                    script{
                         def consoleOutput = currentBuild.rawBuild.getLog(100) // Get last 100 lines of console log
                        mail to: 'recipient@example.com',
                        subject: "Jenkins Build Failed - ${env.JOB_NAME} - #${env.BUILD_NUMBER}",
                        body: """
                        Jenkins build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed.

                        Check console output at ${env.BUILD_URL} to view the full results.

                        Last 100 lines of console output:

                        ${consoleOutput}
                     """
                    }
                }
            } 
        }
    }
}
