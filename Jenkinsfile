node{
 try{
 def mavenHome = tool name: 'maven3.8.5'
 echo "The Job name is: ${env.JOB_NAME}"
 echo "The Build number is: ${env.BUILD_NUMBER}"
 echo "The node name is: ${env.NODE_NAME}"
 
 //POLL SCM
 properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), 
 [$class: 'JobLocalConfiguration', changeReasonComment: ''], 
 pipelineTriggers([pollSCM('* * * * *')])])
  
 //checkout code state
 stage('CheckoutCode'){
 SendSlackNotification('STARTED')
 git branch: 'development', credentialsId: '0223118f-5ede-4965-b0c5-147ec26c84fa', url: 'https://github.com/citibank-gitapps/maven-web-application.git'
 }
 
 //Build
 stage('Build'){
 sh "${mavenHome}/bin/mvn clean package"
 }
 
 //Execute Sonarqube Report
 stage('ExecuteSonarqubeReport'){
 sh "${mavenHome}/bin/mvn sonar:sonar"
 }
 
 //Upload Artifact Into Nexus
 stage('UploadArtifactIntoNexus'){
 sh "${mavenHome}/bin/mvn deploy"
 }
 
 //Deploy Application Into Tomcat Server
 stage('DeployApp'){
 sshagent(['87e7139d-1e3a-4eaa-936f-5ea6958f801b']) {
   sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.32.42:/opt/apache-tomcat-9.0.68/webapps"
}
 }
 }//try Closing 
 catch(e){
 currentBuild.result = "FAILURE"
 }
 finally{
 SendSlackNotification(currentBuild.result)
 }
  
 }//node closing
//Function for Slack Notification
 
 def SendSlackNotification(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'ORANGE'
    colorCode = '#FFA500'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
