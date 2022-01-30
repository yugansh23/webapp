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
	  stage ('Check-Git-Secrets') {
		    steps {
	        sh 'rm trufflehog || true'
		sh 'docker pull gesellix/trufflehog'
		sh 'docker run -t gesellix/trufflehog --json https://github.com/devopssecure/webapp.git > trufflehog'
		sh 'cat trufflehog'
	    }
	    }
	    
	    stage ('Source-Composition-Analysis') {
		steps {
		     sh 'rm owasp-* || true'
		     sh 'wget https://raw.githubusercontent.com/yugansh23/JavaVulnerableLab/master/owasp_dependency_check.sh'	
		     sh 'chmod +x owasp_dependency_check.sh'
		     sh 'bash owasp_dependency_check.sh'
		     sh 'cat /home/jenkins/OWASP-Dependency-Check/reports/dependency-check-report.xml'
		}
	} 
	    stage ('SAST') {
		steps {
		withSonarQubeEnv('sonar') {
			sh 'mvn sonar:sonar'
			sh 'cat target/sonar/report-task.txt'
		       }
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
                    sh 'scp -o StrictHostKeyChecking=no /home/jenkins/workspace/WebApp-pipeline/target/WebApp.war  /var/lib/tomcat9/webapps'
              }      
           }       
    }
	stage ('DAST') {
		      	steps {
			         sh 'docker run -p 8090:8080 -t owasp/zap2docker-stable zap-baseline.py -t http://172.17.0.1:8081/WebApp/ || true'
			   }
		}    
	  stage ('Nikto Scan') {
		    steps {
			sh 'rm nikto-output.xml || true'
			sh 'docker pull secfigo/nikto:latest'
			sh 'docker run --user $(id -u):$(id -g) --rm -v $(pwd):/report -i secfigo/nikto:latest -h 172.17.0.1 -p 8081 -output /report/nikto-output.xml'
			sh 'cat nikto-output.xml'   
		    }
	    }   
	    stage ('SSL Checks') {
		    steps {
			sh 'pip install sslyze==5.0.2'
			sh 'python3 -m sslyze 172.17.0.1:8081 --json_out sslyze-output.json || true'
			sh 'cat sslyze-output.json'
		    }
	    }
     }
}
