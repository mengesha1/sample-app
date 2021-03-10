pipeline{
    agent any
     tools {
    maven 'Maven3'
    }
  stages{
    stage ('checkout'){
        steps {
       checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/mengesha1/sample-app.git']]])  
    }
  }
  stage('Build'){
        steps {
         sh 'mvn clean install -f MyWebApp/pom.xml'
       }
    }   
 stage ('Code Quality Scan') {
      steps {
        withSonarQubeEnv('SonarQube') {
        sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
        }
      }
    }
    stage ('JaCoCo') {
      steps {
      jacoco()
      }
    }
    stage ('Nexus Upload') {
      steps {
    nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: 'f09818ac-0466-487f-a14a-941ed0cb8c5f', groupId: 'myGroupId', nexusUrl: 'http://34.229.67.183:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT '
    }  
   }
    stage ('DEV Deploy') {
      steps {
      echo "deploying to DEV Env "
      deploy adapters: [tomcat9(credentialsId: '6f914f84-baed-48f5-8813-073e306c6ebb', path: '', url: 'http://3.86.18.11:8080')], contextPath: null, war: '**/*.war'
      }
    }
    stage ('Slack Notification') {
      steps {
        echo "deployed to DEV Env successfully"
        slackSend(channel:'menge-jenkins', message: "Job is successful, here is the info - Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
      }
    }
    stage ('DEV Approve') {
      steps {
      echo "Taking approval from DEV Manager for QA Deployment"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to deploy?', submitter: 'admin'
        }
      }
    }
     stage ('QA Deploy') {
      steps {
        echo "deploying to QA Env "
        deploy adapters: [tomcat9(credentialsId: '6f914f84-baed-48f5-8813-073e306c6ebb', path: '', url: 'http://3.86.18.11:8080')], contextPath: null, war: '**/*.war'
        }
    }
    stage ('QA Approve') {
      steps {
        echo "Taking approval from QA manager"
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
        }
      }
    }

  }    
}

