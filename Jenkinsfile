pipeline {
  
  agent any

  tools {
  maven 'maven'
  } 

  environment {
    artifactId = readMavenPom().artifactId()
    groupId = readMavenPom().getGroupId()
    version = readMavenPom().getVersion()
    name = readMavenPom().getName()
  }

  stages {
      
      stage('1. Git CheckOut'){

        steps {
           checkout([
               $class: 'GitSCM', 
               branches: [[name: '*/master']], 
               doGenerateSubmoduleConfigurations: false, 
               extensions: [], 
               submoduleCfg: [], 
               userRemoteConfigs: [[credentialsId: '702a5c2d-3008-4cee-a2a2-ac9305148549', 
               url: 'https://github.com/vinayashokpatil/Jenkins-AWS-Lab.git']]])
        }

      }

      stage('2. Build and package using Maven'){
        
        steps {
           sh 'mvn clean package'
        }


      }

      stage('3. Publish to maven repository - nexus'){
        
        steps {
           nexusArtifactUploader artifacts: [[
             artifactId: '${artifactId}', 
             classifier: '', 
             file: 'target/*.war', 
             type: '*.war']], 
             credentialsId: 'nexus3', 
             groupId: '${groupId}', 
             nexusUrl: '3.137.187.30:8081', 
             nexusVersion: 'nexus3', 
             protocol: 'http', 
             repository: 'devops-aws-lab-RELEASE', 
             version: '${version}'
        }


      }




  }













}