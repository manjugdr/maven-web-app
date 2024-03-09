pipeline {
    agent any
    environment {
        KUBECONFIG = '/var/lib/jenkins/workspace/k8s/' // Specify the path to your Kubernetes configuration file
    }
    stages{
        stage('Build Maven'){
            steps{
                git url:'https://github.com/manjugdr/maven-web-app/', branch: "master"
                sh 'mvn clean package'
                }
        }
        stage('Sonatype Nexus Repository'){
            steps{
            nexusArtifactUploader artifacts: [[artifactId: '01-maven-web-app', classifier: '', file: 'target/01-maven-web-app.war', type: 'war']], credentialsId: 'nexus3', groupId: 'in.ashokit', nexusUrl: '54.82.229.178:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maveen', version: '1.0'   
            }
        }
           stage('Build docker image'){
            steps{
                    script{
                       sshagent(['sshkeypair']) {
                       sh "ssh -o StrictHostKeyChecking=no ubuntu@3.90.191.95"
                       sh 'scp -i chaithra.pem -r /var/lib/jenkins/workspace/maven-web/ ubuntu@172.31.29.59:/home/ubuntu/maven-web-app'
                       sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.29.59"
                       sh 'docker build -t manjugdr/maven-web-app:v1 .'
                }
            }
        }
         }
        stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://3.90.191.95:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd /var/lib/jenkins/workspace/maven-web && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
        stage('Docker login') {
            steps {
                sshagent(['sshkeypair']) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.29.59"
                withCredentials([usernamePassword(credentialsId: 'dockerhub-pwd', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                    sh "echo $PASS | docker login -u $USER --password-stdin"
                    sh 'docker push manjugdr/maven-web-app:v1'
                }
            }
        }
          }
       stage('Deploying App to Kubernetes') {
      steps {
        script {
            sshagent(['sshkeypair']) {
                       sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.24.37"
                sh 'scp -i chaithra.pem  /var/lib/jenkins/workspace/maven-web/deploymentservice.yaml  ubuntu@172.31.24.37:/home/ubuntu/'
          kubernetesDeploy(configs: "deploymentservice.yaml", kubeconfigId: "kubernetes")               
                }
            }
        }
    }
}
}
