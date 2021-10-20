pipeline {
    agent any
    tools { 
        maven 'Maven' 
    }
    stages {
        stage ('Initialize') {
            steps {
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                ''' 
            }
        }
	    
	  stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        } 
      stage ('Deploy-To-Tomcat') {
            steps {
		    sshagent(['tomcat']){
                    sh 'scp -o StrictHostKeyChecking=no /var/lib/jenkins/workspace/CICDPipeline/target/JavaVulnerableLab.war  /opt/tomcat/webapps/webapp.war'
              }      
           }       
    }
	    
     }
}
