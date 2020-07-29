pipeline {
    agent any
  /*     
    tools { 
        maven 'maven' 
        jdk 'jdk1.8' 
    } 
    */
   stages {    
        
      stage('SCM') {
         steps {
            git 'https://github.com/Umeshfarrow/shopizer.git'
         }
      } 
        
      stage('Build Project') {
         steps {
            sh '''
            echo 'cd shopizer'
            mvn clean install
            '''
         }
      }
        
      stage('SonarQube') { 
         steps { 
            sh ''' 
            echo "SonarQube"
            '''   	
         }    
      } 
        
    stage('Docker Image'){
        steps{
            sh '''
            cd sm-shop
            sudo docker build -t umeshfarrow/shopizer .
             '''
           }
    }
    stage('Push Images to repository'){
        steps{
            withCredentials([string(credentialsId: 'Docker_repository', variable: 'dockerPassword')]) {
                sh "sudo docker login -u umeshfarrow -p${dockerPassword}"
            }
            sh 'echo "Push Image to repository" '
            sh 'sudo docker push umeshfarrow/shopizer'
        }
    }
     stage('Approval') {
         // no agent, so executors are not used up when waiting for approvals
         agent none
         steps {
             script {
                 def deploymentDelay = input id: 'Deploy', message: 'Deploy to production?', submitter: 'rkivisto,admin', parameters: [choice(choices: ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '24'], description: 'Hours to delay deployment?', name: 'deploymentDelay')]
                 sleep time: deploymentDelay.toInteger(), unit: 'HOURS'
             }
         }
     }
        stage('AWS Connection and Deployment'){
            steps{
                   sh 'AWS Connection and Deployment'
                   sshagent(['AWS_Deployment_Server']) {
                   sh "ssh -o StrictHostKeyChecking=no ubuntu@34.207.233.199 sudo docker-compose up -d"
                   /** 
                   sh "ssh -o StrictHostKeyChecking=no ubuntu@34.207.233.199 sudo kubectl apply -f <filename> "
                   **/
                   }
            }
        }
        
   } 
   post {
        always {
            //archiveArtifacts artifacts: 'generatedFile.log', onlyIfSuccessful: true
          
            emailext attachLog: true,
                body: "${currentBuild.currentResult}: Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}\n More info at: ${env.BUILD_URL}",
                recipientProviders: [developers(), requestor()],
                subject: "Jenkins Build :- ${currentBuild.currentResult}: Job ${env.JOB_NAME}"
            
        }
    }
}
