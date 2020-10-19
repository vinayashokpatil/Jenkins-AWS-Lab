pipeline {
  
  agent any

  tools {
  maven 'maven'
  } 

  environment {
    ArtifactId = readMavenPom().getArtifactId()
    GroupId = readMavenPom().getGroupId()
    Version = readMavenPom().getVersion()
    VersionCheck = readMavenPom().getVersion().endsWith("-SNAPSHOT")
    
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

      stage ('Print') {

        steps {
           sh "echo ${ArtifactId}"
           sh "echo ${GroupId}"
           sh "echo ${Version}"
           sh "echo ${VersionCheck}"
      }
      }

      stage('3. Publish to maven repository - nexus'){
        
         steps {

           script {

             def NexusRepo = Version.endsWith("-SNAPSHOT") ? "Jenkins-AWS-Lab-SNAPSHOT" : "Jenkins-AWS-Lab-RELEASE"

             nexusArtifactUploader artifacts: [[
             artifactId: "${ArtifactId}", 
             classifier: '', 
             file: "target/${ArtifactId}-${Version}.war", 
             type: 'war']], 
             credentialsId: 'nexus3', 
             groupId: "${GroupId}", 
             nexusUrl: '172.31.9.39:8081', 
             nexusVersion: 'nexus3', 
             protocol: 'http', 
             repository: "${NexusRepo}", 
             version: "${Version}"
          }
        }
     }
   
     stage ('4.Transfer to STAGING folder if the version tag is RELEASE else transfer to TEST folder') {

       steps {
         script {
           def VersionTag = Version.endsWith("-SNAPSHOT") ? "//opt//playbooks//artifacts-from-jenkins-TEST" : "//opt//playbooks//artifacts-from-jenkins-STAGING"
           
           sshPublisher(publishers: 
           [sshPublisherDesc(
             configName: 'ansible_controller_instance', 
             transfers: [
               sshTransfer(
                 cleanRemote: false, 
                 excludes: '', 
                 execCommand: '', 
                 execTimeout: 120000, 
                 flatten: false, 
                 makeEmptyDirs: false, 
                 noDefaultExcludes: false, 
                 patternSeparator: '[, ]+', 
                 remoteDirectory: "${VersionTag}", 
                 remoteDirectorySDF: false, 
                 removePrefix: 'target', 
                 sourceFiles: 'target/*.war')], 
                 usePromotionTimestamp: false, 
                 useWorkspaceInPromotion: false, 
                 verbose: false)])
         }
       }
     }

     stage ('5. Invoke Ansible Playbook to DEPLOY on tomcat based on STAGING/TEST') {
       steps {
         script {
            
          def Command = Version.endsWith("-SNAPSHOT") ? "ansible-playbook /opt/playbooks/deploywar-to-tomcat_TEST.yaml -i /opt/playbooks/hosts" : "ansible-playbook /opt/playbooks/deploywar-to-tomcat_STAGING.yaml -i /opt/playbooks/hosts"
          
          sshPublisher(publishers: 
          [sshPublisherDesc(
            configName: 'ansible_controller_instance', 
            transfers: [
              sshTransfer(
                cleanRemote: false, 
                excludes: '', 
                execCommand: "${Command}", 
                execTimeout: 120000, 
                flatten: false, 
                makeEmptyDirs: false, 
                noDefaultExcludes: false, 
                patternSeparator: '[, ]+', 
                remoteDirectory: '//opt//playbooks', 
                remoteDirectorySDF: false, 
                removePrefix: '', sourceFiles: '')], 
                usePromotionTimestamp: false, 
                useWorkspaceInPromotion: false, 
                verbose: false)])
         }
       }
     }



  }

}