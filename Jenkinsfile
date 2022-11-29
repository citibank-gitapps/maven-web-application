node{
 def mavenHome = tool name: 'maven3.8.5'
 echo "The Job name is: ${env.JOB_NAME}"
 echo "The Build number is: ${env.BUILD_NUMBER}"
 echo "The node name is: ${env.NODE_NAME}"
 
 properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), 
 [$class: 'JobLocalConfiguration', changeReasonComment: ''], 
 pipelineTriggers([pollSCM('* * * * *')])])
  
 //checkout code state
 stage('CheckoutCode'){
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
 
  
 }//node closing
