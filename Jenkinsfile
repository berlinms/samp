
node {
   def props = readProperties file: "${workspace}/gradle.properties"
   env.artifactId = props.artifactId 
   env.groupId = props.groupId
   env.version = props.version
   env.filePath = props.filePath
   env.tag = props.tag
   env.credentialsId = props.credentialsId
   env.usernameVariable = props.usernameVariable
   env.passwordVariable = props.passwordVariable
   env.tag_url = props.tag_url
   env.nexusInstanceId = props.nexusInstanceId
   env.nexusRepositoryId = props.nexusRepositoryId
   env.packaging = props.packaging
   env.tag = props.tag
   env.email = props.email

}

pipeline {
    agent any



    stages {

         stage('checkout') {

            steps {       
            checkout scm 
         }
           
         }


          stage('Compile') {
            steps {

            
              sh 'gradle compileTestJava'
               
                  }
           }


           stage('Create war') {
             steps {
                sh 'gradle war'
                 }
           }
        
          stage('SonarQube analysis') { 
            steps {    
                 withSonarQubeEnv("local") { // Will pick the global server connection you have configured
                      sh 'gradle sonarqube'
               }
              }

           }
          stage("publish")  {
           
            steps {

            
           nexusPublisher nexusInstanceId: env.nexusInstanceId, nexusRepositoryId:env.nexusRepositoryId , packages: [[$class: 'MavenPackage', mavenAssetList: [[classifier: '', extension: '', filePath: env.filePath ]], mavenCoordinate: [artifactId:env.artifactId , groupId:env.groupId , packaging: env.packaging, version:"${env.version}.${BUILD_NUMBER}" ]]]
             }
      
             }  
          stage('tag create') {
             steps {
            
                sh("git tag  ${JOB_NAME}.${BUILD_NUMBER}")
                 }
           } 
 	  stage("tag push")  {
           
            steps {
	   withCredentials([[$class: 'UsernamePasswordMultiBinding', 
                credentialsId: env.credentialsId, 
                usernameVariable:env.usernameVariable , 
                passwordVariable:env.passwordVariable ]]) {
		   sh("git push  ${env.tag_url} ${JOB_NAME}.${BUILD_NUMBER}")
		}
              }
          }
           
	 stage("cucumber")  {
		   
		    steps {
		     cucumber failedFeaturesNumber: -1, failedScenariosNumber: -1, failedStepsNumber: -1, fileIncludePattern: '**/*.json', pendingStepsNumber: -1, skippedStepsNumber: -1, sortingMethod: 'ALPHABETICAL', undefinedStepsNumber: -1
	      }
	      
	  } 

	
	stage('bu') {
            
             steps {

         
        echo "cleanup workspace"
        deleteDir()

       

        walk job: 'poll', jobAction: '''
            dir(samp) {
                git url: git@github.com:jenkinsci/pipeline-dependency-walker-plugin.git, branch: master
                withMaven(maven: 'mvn', mavenLocalRepo: mvnRepo) {
                    sh "mvn clean install"
                }
            }
        '''
    }
  }


}
    
    post {

        failure {
            mail to: env.email,
            subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            body: "Build failed: ${env.BUILD_URL}"
        }

        success {
           script {
            if (currentBuild.previousBuild != null && currentBuild.previousBuild.result != 'SUCCESS') {
                mail to: env.email,
                subject: "Pipeline Success: ${currentBuild.fullDisplayName}",
                body: "Build is back to normal (success): ${env.BUILD_URL}"     
            } 
           }          
        }
    }      
}



