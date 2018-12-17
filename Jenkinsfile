pipeline {
 agent {node {label 'master'}}
 environment {
  dotnet = "${tool 'MSBuild15_Path'}"
 }
 stages {
  stage('Checkout') {
   steps {
    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '21dcef66-4390-4b26-8345-bdce1adc6a5d', url: 'https://github.com/rohithks/dotnet-project.git']]])
   }
  }
  
  stage('Load Properties') {
  steps {
    load "helloworldjenkins.properties"
  }
  }
	 
   stage('Restore Packages') {
   steps {
     bat "echo ${APP_NAME}"
      script {
            if ("${APP_NAME}" == 'HelloWorld') {
            echo 'Restoring the packages for Helloworld'
            bat "\"${env.Nuget_Path}\" restore 1-hello-world"
             } else {
            echo 'No Packages for helloworld'
     }   
   }  
  }
 }	
   stage ('Build & SonarAnalysis'){
   steps {
      script {
            if ( "${MS_BUILD}" == 'YES'  &&  "${SONAR_ANALYSIS}" == 'YES' ) {  
                 withCredentials([string(credentialsId: 'Sonarqube_Token', variable: 'SonarqubeToken')]) {					
               bat "\"${tool 'SonarQube_MSBuild'}\\SonarScanner.MSBuild.exe\" begin /k:\"HelloWorld\" /d:sonar.host.url=" + env.SonarQube_URL + " /d:sonar.login=${SonarqubeToken}"
				
               bat "\"${tool 'MSBuild15_Path'}\\msbuild.exe\" 1-hello-world\\1-hello-world.sln /t:Rebuild /p:DeployOnBuild=true /p:PackageAsSingleFile=true /p:platform=\"any cpu\" /p:configuration=\"release\""
				
               bat "\"${tool 'SonarQube_MSBuild'}\\SonarScanner.MSBuild.exe\" end /d:sonar.login=${SonarqubeToken}" }
               
	      } else {
               echo 'No Build and Sonaranalysis'
        }
      }
    }
   }
  
   stage('Pack') {
   steps {
      script {
          if ("${NUPKG_PACK}" == 'YES') {
              bat "\"${env.Nuget_Path}\" pack 1-hello-world"
	  } else {
              echo 'Not packing anything'
    }
   }
   }
  }
  
   stage('Artifactory Upload') {
   steps {
      bat "echo ${env.Nupkg_Path}"
      bat move c:\Sourcefoldernam\*.* e:\destinationFolder
      script { 
	  def buildVersion = currentBuild.number
          def server = Artifactory.server 'Jfrog_Artifactory'
          def buildInfo = Artifactory.newBuildInfo()
          buildInfo.name = "${BUILD_INFO_NAME}"
          buildInfo.number = "${BUILD_INFO_NUMBER}"
          server.publishBuildInfo buildInfo
                def uploadSpec = """{
                    "files": [
		    {
                       "pattern":"${env.Nupkg_Path}/1-hello-world.1.0.0.nupkg",
                       "target": "Nuget-repo-test/${JIRA_STORY_ID}/"
                    }
		    ]
                 }"""
                  server.upload(uploadSpec) 
		  //def buildInfo1 = server.download downloadSpec
                  def buildInfo2 = server.upload uploadSpec
                  //buildInfo1.append buildInfo2
                  server.publishBuildInfo buildInfo2
                  
	 }
      }
    }
  
  stage('Artifactory Download') {
  agent {node {label 'AnsibleSlave1'}}
  steps {
      script { 
	  def buildVersion = currentBuild.number
          def server = Artifactory.server 'Jfrog_Artifactory'
          def buildInfo = Artifactory.newBuildInfo()
          buildInfo.name = "${BUILD_INFO_NAME}"
          buildInfo.number = "${BUILD_INFO_NUMBER}"
          server.publishBuildInfo buildInfo
                def downloadSpec = """{
                    "files": [
		    {
		       "pattern":"Nuget-repo-test/${JIRA_STORY_ID}/",
                       "target": "${env.Nupkg_Path_Slave}/"
                    }
		    ]
                 }"""
                  server.download(downloadSpec) 
		  def buildInfo1 = server.download downloadSpec
                  //def buildInfo2 = server.upload uploadSpec
                  //buildInfo1.append buildInfo2
                  server.publishBuildInfo buildInfo1       
	 }
      }
    }
  }
}


  
