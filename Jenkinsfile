#!/usr/bin/env groovy

node{
  def workspace = env.WORKSPACE
  def sparseDir = '/home/dhirendra/eclipse-workspaces/spring-boot-web-jsp'
  
  stage('clean-workspace') {
   //cleanWs()
   echo "current working directory is : " 
   echo pwd

   echo "workspace directory is ${workspace}"
   echo "${sparseDir}"
   // echo  "${sparseDir} >> ${workspace}/.git/info/sparse-checkout"
   //sh 'rm -rf ${workspace}/*'

   sh 'java -version' //working
   sh 'mvn --version' //working

   maven 'mvn clean'
   echo "after maven clean"
    }
    stage('scm') {
      echo 'scm sparse checkout3'
      
      echo "workspace directory is ${workspace}"
  
      //def rtGradle = Artifactory.newMavenBuild()
      
      def config = [:]
   
      config.serviceRepo = 'https://github.com/dhir007/spring-boot-web.git'
      config.sparseCheckout= '/home/dhirendra/eclipse-workspaces/spring-boot-web-jsp'
      config.gitBranch = 'master' 
      //config.serviceRepo = 'https://github.com/rbhupesh/techinfo.git'
          //config.gitBranch = 'master'   
   
      echo "check out branch ${config.gitBranch} from ${config.serviceRepo}"                       
      git branch: config.gitBranch, url: config.serviceRepo //working step
   
    }
    stage('mvn-build') {
      echo "mvn-build"
      echo "workspace directory is ${workspace}"
      echo "${sparseDir}"
      echo '${workspace}/${sparseDir}'
      //sh "cd ${workspace}/${sparseDir}"
      sh "cd ${workspace}"
      //maven 'mvn clean compile test package'
      // Create an Artifactory Maven instance.
      //def rtMaven = Artifactory.newMavenBuild()
      //def buildInfo
      //buildInfo = rtMaven.run pom: 'maven-example/pom.xml', goals: 'clean package'
  
      echo "current working directory is : " 
      sh "pwd"
      //sh "mvn -f  ${workspace}/${sparseDir}/pom.xml clean compile package"// running locally 
      sh "mvn -f  ${workspace}/pom.xml clean compile package"// running locally    
    }
    stage('mvn-junit') {
      echo 'mvn-junit'
    }
    stage('mvn-sonar') {
      echo "mvn-sonar"      
      //sh "printenv"
      //echo "after printENV"
   
      withSonarQubeEnv("localSonarQube") {
                        sh "mvn -f  ${workspace}/pom.xml sonar:sonar"// running locally
      }
      //sh "mvn -f  ${workspace}/pom.xml sonar:sonar"// running locally
    }
    stage("Quality-Gate1"){
    timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
      def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
       if (qg.status != 'OK') {
          error "Pipeline aborted due to quality gate failure: ${qg.status}"
         }
    }
  }
  stage("push-to-pcf"){
    sh "cf login api https://api.run.pivotal.io -u dhir28@gmail.com -p Pivotel@2018 -o dhir-org -s development"
    sh "cf push"
  }
  stage("Jmeter-p1"){
  //sh "cd ${workspace}/${sparseDir}/src/test/jmeter"
  sh "ls -l"
  sh "/home/dhirendra/apache-jmeter-4.0/bin/jmeter -Jjmeter.save.saveservice.output_format=xml -n -t /home/dhirendra/eclipse-workspaces/SpringBootUnitTestTutorial/Thread_Group.jmx -l Test.jtl"
  }

}
