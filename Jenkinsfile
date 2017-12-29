#!/bin/bash +x
//TODO - Make SVN and GIT Checkout steps perfect with Jenkins way. Do not use Shell way.

def temporaryDockerRegistry = tempDockerRegistry+":443"
def permanentDockerRegistry = permDockerRegistry+":443"
def nexusRepoHostPort = nexusRepositoryHost
def nexusRepo = nexusRepository
def testDevelopmentIp="172.19.74.225"
// This update is for Bug ID : 531
def httpProxy = ''
def httpsProxy = ''
  
node {
  echo "Parameters"
  echo "Temp Docker Registry: ${tempDockerRegistry}"
  echo "Permenant Docker Registry: ${permDockerRegistry}"
  echo "Docker Repository: ${dockerRepo}"
  echo "Docker Image Name: ${dockerImageName}"
  

  
  echo "SCM Type: ${scmSourceRepo}"
  echo "SCM Path: ${scmPath}"
  echo "SCM User: ${scmUsername}"
  echo "MEC User: ${userId}"
  
  echo "HTTP Proxy: ${httpProxy}"
  echo "HTTPS Proxy: ${httpsProxy}" 

  echo "Dgoss Test File: ${dgossFile}" 
   
  
  //-------------------------------------- big bucket
  //To escape all Special Charecters in a given input string password
  def pwdstr = scmPassword
  scmPassword = pwdstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def usrstr = scmUsername
  scmUsername = usrstr.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
  
  def pwdstr2 = scmPassword
  def usrstr2 = scmUsername
  scmPassword = pwdstr2.replaceAll( /([@])/, '%40' )
  scmUsername = usrstr2.replaceAll( /([@])/, '%40' ) 
  defGossFile = dgossFile
  //----------------------------------------

  stage('Code Pickup') {
    //slackSend "Code Pickup started for ${env.JOB_NAME} ${env.BUILD_NUMBER} for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 

    echo "Source Code Repository Type : ${scmSourceRepo}"
    echo "Source Code Repository Path : ${scmPath}"
    
    if("${scmSourceRepo}".toUpperCase()=='SVN'){
        //Not a perfect solution. Reimplement with SVN Step or checkout Step
        echo 'SCM is SVN'
        sh "svn co --username ${scmUsername} --password ${scmPassword} ${scmPath} ."
        
    } else if("${scmSourceRepo}".toUpperCase()=='GIT' || "${scmSourceRepo}".toUpperCase()=='GITHUB'){
        //Not a perfect Solution. Reimplement with Git Step or checkout Step
        echo 'SCM is GIT Based'
        //Check if the Git clone URL already has username and password in it.
        //def strToMatch = "${scmPath}"
        def extractedCredentialsFromPath="";
        if(scmPath.indexOf("@")>-1 && scmPath.indexOf("//") >-1) {
            extractedCredentialsFromPath=scmPath.substring(scmPath.indexOf("//")+2,scmPath.indexOf("@"))
        }
      
        if(extractedCredentialsFromPath && extractedCredentialsFromPath.length()>0) {
          echo "Looks like the Username or Password or Both are found in the GIT Repo path itself"
          extractedCredentialsFromPath=extractedCredentialsFromPath.replaceAll( /([^a-zA-Z0-9])/, '\\\\$1' )
          extractedCredentialsFromPath=extractedCredentialsFromPath.replaceAll( /([@])/, '%40' )
          echo "Extracted Credentials and ecapped: ${extractedCredentialsFromPath}"
          echo "User & Password: ${scmUsername}\\:${scmPassword}"
          if ("${extractedCredentialsFromPath}" == "${scmUsername}" + "\\:" + "${scmPassword}") {
            echo "Username and password already present in url. No need to append"
          } else if("${extractedCredentialsFromPath}" == "${scmUsername}" && !scmPath.startsWith("ssh://")) {
            echo "Username already present in url. Just handle the password" 
            scmPath = scmPath.substring(0,scmPath.indexOf("@"))+":" + scmPassword +scmPath.substring(scmPath.indexOf("@"), scmPath.length());
          } else if("${extractedCredentialsFromPath}" == "${scmUsername}" && scmPath.startsWith("ssh://")) {
            echo "Username and password already present in url. Since this is SSH, no need to append password" 
            //Password should not be appended for SSH. It should be passed through SSHPASS only
          } else {
            echo "Something wrong. The credentials in the URL did not match with the values passed" 
          }
        } else {
          echo 'Credentials not found in path. Frame it'
          //SSH based git should not append password. It will not work. The password should be passed through sshpass
          if(scmPath.startsWith("ssh://")){
              scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
          } else {         
              scmPath = scmPath.substring(0, scmPath.indexOf("//")+2) + scmUsername + ":" + scmPassword + "@" +scmPath.substring(scmPath.indexOf("//")+2, scmPath.length());
          }  
        }
      
        //echo "GIT PATH: ${scmPath}"
        try {
            //If we use git clone, it will not clone in the same path if we rebuild the pipeline
            sh 'ls -a | xargs rm -fr'
        } catch (error) {
        }
      
        if(scmPath.startsWith("ssh://")){
          if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0){
            echo "Looks like this Jenkins behind Proxy"
            sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && sshpass -p ${scmPassword}   git clone --recursive ${scmPath} ."
          } else {
            echo "Looks like this Jenkins is not behind Proxy"
            sh "sshpass -p ${scmPassword}   git clone --recursive ${scmPath} ."
          }            
        } else {
            //Need this line for the recursive sub module clone to work without asking credentials
            sh "git config --global credential.helper 'cache --timeout=120'"
            if(httpsProxy != null && httpProxy!=null && httpsProxy.length()>0 && httpProxy.length()>0) {
              echo "Looks like this Jenkins behind Proxy"
              sh "export https_proxy=${httpsProxy} && export http_proxy=${httpProxy} && git clone --recursive ${scmPath} ."
            } else {
              echo "Looks like this Jenkins is not behind Proxy"
              sh "git clone --recursive ${scmPath} ."
            }  
            //Need this line for the recursive sub module clone to work without asking credentials -- Clear the cache. 
            sh "git credential-cache exit"
        } 
        
        //The below solutions not working with username and password
        //git "${scmPath}" //Alternate option
        //checkout scm: [$class:'GitSCM', userRemoteConfigs: [[url: scmPath ]]]
    } else {
        error 'Unknown Source code repository. Only GIT and SVN are supported'
    }
    //slackSend "Code Pickup finished for ${env.JOB_NAME} ${env.BUILD_NUMBER} for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 
  } 
  //---------------------------------------
  
  //Preparing for Build & Package
  //slackSend "Dockerization started for ${env.JOB_NAME} ${env.BUILD_NUMBER} for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 
  def appModuleSeperated = fileExists 'app'
  def testModuleSeperated = fileExists 'test'
  def appPath = ''
  def testPath = ''  
  if (appModuleSeperated) {
    echo 'There is a directory called app and hence assuming that /app is the working directory for application.'
    appPath='app/'
  } else {
    echo 'There is no directory named app. Hence assuming that the current directory is the working directory.'
    appPath = ''
  }

  if (testModuleSeperated) {
    echo 'There is a directory called test and hence assuming that /test is the integration test automation suite.'
    testPath = 'test/'  
  } else {
    echo 'There is no directory named test. Hence assuming that there is no integration testing automation suite.'
    testPath = ''  
  }
  
  def isBuildAndPackageRequired = true
  def isTestPackageRequired = false;   
  def buildDockerFile = appPath + 'Dockerfile.build'
  def distDockerFile = appPath + 'Dockerfile.dist'
  def testDockerFile = appPath + 'Dockerfile.test'
  if (fileExists(buildDockerFile) && fileExists(distDockerFile)) {
    echo 'It looks like this application is compiler based application and hence there is a seperate dockerfile found for compile, build and packaging.'
    isBuildAndPackageRequired = true;    
  }else if (appPath + fileExists('Dockerfile')) {
    echo 'It looks like there is only one docker file. May be this is interpreter based application technology.'
    isBuildAndPackageRequired = false;
    distDockerFile = appPath + 'Dockerfile'
  } else {
    echo 'Dockerfile not found under ' + appPath
  }

  if(fileExists(testDockerFile)){
      echo 'A test dockerfile is also found. Application will be tested based on the test dockerfile.'
      isTestPackageRequired = true;   
  }else {
      echo 'A test dockerfile was not found. Application will  not be tested in a sandbox.'
      isTestPackageRequired = false;   
  }
 
  def appWorkingDir = (appPath=='') ? '.' : appPath.substring(0, appPath.length()-1)
  //End of Preparation for Build and Package
  
  //TODO - Tune it later. Dirty solution to identify the Jenkins generated artifacts for Nexus
        sh 'echo Nexus>Nexus.txt'
  //Dirty solution ends.
  
  //---------------------------------------
  if(isBuildAndPackageRequired){
    echo 'Compile and Package runs as seperate stage due to this app requirements.'
    stage('Compile, Unit Test & Package') {
      echo 'Working Directory for Docker Build file: ' + appWorkingDir
      echo "Build Tag Name: ${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER}"
      echo "Build params: --file ${buildDockerFile} ${appWorkingDir}"
      
      appCompileAndPackageImg = docker.build("${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER}", "--file ${buildDockerFile} ${appWorkingDir}")      
      
      //Reading the CMD from Docker file and would be executing within the container. This is due to the behaviour of this plugin
      //TODO - Danger zone. This approach is not based on grammer. So in case if the CMD is not shell (if exec) or ENTRYPOINT is given, this approach would not work
      def dockerCMD = readFile buildDockerFile
      echo dockerCMD.substring(dockerCMD.indexOf('CMD')+3, dockerCMD.length())      
      appCompileAndPackageImg.inside('--net=host') {        
        sh dockerCMD.substring(dockerCMD.indexOf('CMD')+3, dockerCMD.length())
      }
      //End of Danger Zone code      
    }
  } else {
    echo 'There is no Compile and Package as seperate step'
  }
  //---------------------------------------
  
  if("${stage}".toUpperCase() == 'BUILD') {
    //slackSend "The requested stage is Build only. Hence pushing the successful image into temporary repo for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 
    echo 'The requested stage is Build only. Hence pushing the successful image into temporary repo'
    //---------------------------------------
    // We are pushing to a private Temporary Docker registry as this is just Build case.
    // 'docker-registry-login' is the username/password credentials ID as defined in Jenkins Credentials.
    // This is used to authenticate the Docker client to the registry.
    //docker.withRegistry("http://${temporaryDockerRegistry}/", 'docker-registry-login') {
    //withDockerRegistry([credentialsId: 'docker-registry-login', url: temporaryDockerRegistry]) {
   docker.withRegistry("http://${temporaryDockerRegistry}/", 'docker-registry-login') {
      def pcImg
      stage('Dockerization & Stage') {
        // Let us tag and push the newly built image. Will tag using the image name provided. No need of Docker hostname as it appends itself.
        echo "BUILD TAG USED FOR IMAGE : ${env.BUILD_NUMBER}";
        pcImg = docker.build("${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER}", "--network host --file ${distDockerFile} ${appWorkingDir} ")
        pcImg.push("${env.BUILD_NUMBER}");
      }
      //slackSend "BUILD stage pushed to temporary docker repository." 
    }   
  } else if ("${stage}".toUpperCase() == 'DEPLOY') {
  //slackSend "The requested stage is Deploy. Hence without certifying, the image is pushed to permanent repo for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 
    echo 'The requested stage is Deploy. Hence without certifying, the image is pushed to permanent repo'
    //withDockerRegistry([credentialsId: 'docker-registry-login', url: permanentDockerRegistry]) {
    docker.withRegistry("https://${permanentDockerRegistry}/", 'docker-registry-login') {
      def pcImg
      stage('Dockerization & Publish') {
        // Let us tag and push the newly built image. Will tag using the image name provided. No need of Docker hostname as it appends itself.
        echo "DEPLOY TAG USED FOR IMAGE is always latest";
        pcImg = docker.build("${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER}", "--network host --file ${distDockerFile} ${appWorkingDir} ")
        pcImg.push('latest');
      }
      slackSend "DEPLOY stage pushed to permanent docker repository without certifying."
    }    
  } else if ("${stage}".toUpperCase() == 'CERTIFY'){
  //slackSend "The requested stage is Certify. Hence just publishing to temporary repo and provisioning sandbox for target ${dockerImageName}:${env.BUILD_NUMBER} from ${scmPath}/${scmSourceRepo} (<${env.BUILD_URL}|Open>)" 
    echo 'The requested stage is Certify. Testing the image and pushing into permanent Docker Registry'
    docker.withRegistry("http://${temporaryDockerRegistry}/", 'docker-registry-login') {
      def pcImg
      stage('Certify') {
        // Let us tag and push the newly built image. Will tag using the image name provided. No need of Docker hostname as it appends itself.
        //pcImg = docker.build("${temporaryDockerRegistry}/${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER}", "--file ${distDockerFile} ${appWorkingDir}")
        echo "CERTIFY TAG USED FOR IMAGE : ${env.BUILD_NUMBER}";
        pcImg = docker.build("${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER}", "--network host --file ${distDockerFile} ${appWorkingDir} ")
        pcImg.push("${env.BUILD_NUMBER}");
      }
      //slackSend "CERTIDY stage started. Sandboxing Initiated."
    }   
  }
  //---------------------------------------
  stage('Testing in Container Sandbox') {
    if(isTestPackageRequired){
          echo 'Testing the dockerfile before pushing to permanenet repository.'
          if (fileExists('test.sh')) {
            echo 'Test.sh found'
          }
          else 
          {
            echo 'No test.sh'
          }
          echo 'Working Directory for Docker Build file: ' + appWorkingDir
          echo "Build Tag Name: ${dockerRepo}/${dockerImageName}-build:${env.BUILD_NUMBER}"
          echo "Build params: --file ${testDockerFile} ${appWorkingDir}"
          
          appCompileAndPackageImg = docker.build("${dockerRepo}/${dockerImageName}-sandbox:${env.BUILD_NUMBER}", "--file ${testDockerFile} ${appWorkingDir}")      
          
          //Reading the CMD from Docker file and would be executing within the container. This is due to the behaviour of this plugin
          //TODO - Danger zone. This approach is not based on grammer. So in case if the CMD is not shell (if exec) or ENTRYPOINT is given, this approach would not work
          echo '${dockerImageName}-sandbox:${env.BUILD_NUMBER}'   
          sh "docker run ${dockerRepo}/${dockerImageName}-sandbox:${env.BUILD_NUMBER}"
          //appCompileAndPackageImg.inside('--net=host') 
          //{        
            //sh dockerCMD.substring(dockerCMD.indexOf('CMD')+3, dockerCMD.length())
            //sh './test.sh'
            //End of Danger Zone code   
         // }   
    }
    else{
        echo 'No Dockerfile corresponding to tests have been found.'
    }
  }

  stage('Sanity Testing using dGoss') {
    if("${dgossFile}".toUpperCase() == 'NONE') {
    //slackSend "Developer did not provide a goss.yaml for  ${dockerImageName}:${env.BUILD_NUMBER}. Ignoring this test."
    echo 'The requested stage is dGoss but yaml was not found. Hence aborting the testing and pushing the successful image into temporary repo'
    //---------------------------------------
    // We are pushing to a private Temporary Docker registry as this is just Build case.
    // 'docker-registry-login' is the username/password credentials ID as defined in Jenkins Credentials.
    // This is used to authenticate the Docker client to the registry.
    //docker.withRegistry("http://${temporaryDockerRegistry}/", 'docker-registry-login') {
    //withDockerRegistry([credentialsId: 'docker-registry-login', url: temporaryDockerRegistry]) {
   
  }else {
    sh "mkdir -p /goss/${env.JOB_NAME}-${env.BUILD_NUMBER}"
    sh "ssh root@172.19.74.232 'mkdir -p /root/gosstest/${env.JOB_NAME}/latest'" 
    sh "ssh root@${testDevelopmentIp} 'mkdir -p /root/gosstest/${env.JOB_NAME}/latest'"  
    //slackSend "dGoss testing started for  ${dockerImageName}:${env.BUILD_NUMBER}. "
    echo 'The requested stage is dGoss testing with a YAML file. Hence testing the image pushed to permanent repo'
    echo "DGOSS TESTING TAG USED FOR IMAGE : ${env.BUILD_NUMBER}";
    //sh "cp /goss/goss.yaml ."
    sh "dgoss run ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} > /goss/${env.JOB_NAME}-${env.BUILD_NUMBER}/dGossSanityReport.txt"
    sh "scp /goss/${env.JOB_NAME}-${env.BUILD_NUMBER}/dGossSanityReport.txt  root@172.19.74.232:/root/gosstest/${env.JOB_NAME}/latest/report.txt"
    sh "scp /goss/${env.JOB_NAME}-${env.BUILD_NUMBER}/dGossSanityReport.txt  root@${testDevelopmentIp}:/root/gosstest/${env.JOB_NAME}/latest/report.txt"  
  } 
  //slackSend "dGoss unit testing complete."
  //---------------------------------------

  stage('Anchore Vulnerability Scanning') {
    try{
        sh "mkdir -p /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}"
        sh "ssh root@172.19.74.232 'mkdir -p /anchore/${env.JOB_NAME}/latest'"      
        //sh "ssh root@172.19.74.232 'rm *.* /anchore/${env.JOB_NAME}/latest'"
        echo "The requested stage is Ancore vulnerability scanning testing known CVE for targets."
        sh "docker exec anchore anchore analyze --image ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} --imagetype base > /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/anchore_analysis_report.txt"
        echo "Anchore analysis complete for ${dockerImageName}:${env.BUILD_NUMBER}"
        sh "docker exec anchore anchore audit --image ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} report > /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/anchore_audit_report.txt"
        echo "Anchore audit complete for ${dockerImageName}:${env.BUILD_NUMBER}"
        sh "docker exec anchore anchore query --image ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} list-files-detail all > /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/anchore_files_report.txt"
        echo "Anchore query complete for all files in ${dockerImageName}:${env.BUILD_NUMBER}"
        sh "docker exec anchore anchore query --image ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} cve-scan all > /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/anchore_cve_report.txt"
        echo "Anchore CVE scan complete for all vulnerabilities in ${dockerImageName}:${env.BUILD_NUMBER}"
        sh "docker exec anchore anchore toolbox --image ${dockerRepo}/${dockerImageName}:${env.BUILD_NUMBER} show > /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/anchore_toolbox_show_final.txt"
        echo "The final report is prepared for Jenkins Admin by Anchore Scanner."
        sh "scp /anchore/${env.JOB_NAME}-${env.BUILD_NUMBER}/*.txt  root@172.19.74.232:/anchore/${env.JOB_NAME}/latest"
        emailext attachmentsPattern: '/anchore/${env.JOB_NAME}/*.txt', body: 'Find attachments', subject: 'Anchore Vulnerability Reports', to: 'harsh2.singh@gmail.com'
  //---------------------------------------
    }
    catch(err) { 
            echo 'Anchore test failed.'
            slackSend "Anchore Vulnerability Scanning FAILED for ${env.JOB_NAME} ${env.BUILD_NUMBER} for target ${dockerImageName}:${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)" 

  }
 }
}
  stage('Publish Jenkins Output to Nexus'){
    echo 'Publishing the artifacts...';
    //def PWD = pwd(); //"${PWD}/artifacts.tar.gz"
    //sh 'find . -type f -newer Nexus.txt -print0 | tar -czvf artifacts.tar.gz --ignore-failed-read --null -T -'
    //Fixed for archive overlap issue
    try{
      sh 'find . -type f -newer Nexus.txt -print0 | tar -zcvf artifacts.tar.gz --ignore-failed-read --null -T -' 
    } catch(Exception e) {
    }
    //Nexus 2
    //nexusArtifactUploader artifacts: [[artifactId: "${env.JOB_NAME}", classifier: '', file: 'artifacts.tar.gz', type: 'gzip']], credentialsId: 'Nexus', groupId: 'org.jenkins-ci.main.mec', nexusUrl: '13.55.146.108:8085/nexus', nexusVersion: 'nexus2', protocol: 'http', repository: 'MEC',version: "${env.BUILD_NUMBER}"
    //Nexus 3
    nexusArtifactUploader artifacts: [[artifactId: "${env.JOB_NAME}", classifier: '', file: 'artifacts.tar.gz', type: 'gzip']], credentialsId: 'Nexus', groupId: 'org.jenkins-ci.main.mec', nexusUrl: nexusRepoHostPort, nexusVersion: 'nexus3', protocol: 'http', repository: nexusRepo,version: "${env.BUILD_NUMBER}"
    sh 'rm Nexus.txt'    
    //Dirty solution ends
  }
}
