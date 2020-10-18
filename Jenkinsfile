pipeline {
  
  agent any

  tools {
  maven 'maven'
  } 

  environment {
    ArtifactId = readMavenPom().getArtifactId()
    GroupId = readMavenPom().getGroupId()
    Version = readMavenPom().getVersion()
    Name = readMavenPom().getName().endsWith("-RELEASE")
    Release = ''

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

      stage('3. Publish to maven RELEASE repository - nexus'){
        
        when { environment name: 'Name', value: 'true' }

        steps {

            sh "echo '${Name}'"

           nexusArtifactUploader artifacts: [[
             artifactId: "${ArtifactId}", 
             classifier: '', 
             file: "target/${ArtifactId}-${Version}.war", 
             type: 'war']], 
             credentialsId: 'nexus3', 
             groupId: 'lu.amazon.aws.demo', 
             nexusUrl: '172.31.9.39:8081', 
             nexusVersion: 'nexus3', 
             protocol: 'http', 
             repository: "Jenkins-AWS-Lab-RELEASE", 
             version: "${Version}"
        }
     }

     stage('4. Publish to maven SNAPSHOT repository - nexus'){
        
        
        when { environment name: 'Name', value: 'false' }
        
        steps {

            sh "echo '${Name}'"

           nexusArtifactUploader artifacts: [[
             artifactId: "${ArtifactId}", 
             classifier: '', 
             file: "target/${ArtifactId}-${Version}.war", 
             type: 'war']], 
             credentialsId: 'nexus3', 
             groupId: 'lu.amazon.aws.demo', 
             nexusUrl: '172.31.9.39:8081', 
             nexusVersion: 'nexus3', 
             protocol: 'http', 
             repository: "Jenkins-AWS-Lab-SNAPSHOT", 
             version: "${Version}"
        }
     }


  }

}