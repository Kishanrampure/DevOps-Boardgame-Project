pipeline {
    agent any
 tools {
        maven 'M3'
        jdk 'jdk'
    }
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage('Git Init') {
        steps{
            sh 'git init'
            }
        }
        stage('SCM Fetch') {
        steps{
            git branch: 'main', url: 'https://github.com/Kishanrampure/DevOps-Petclinic-Project.git'
            }
        }
        stage('Build') {
        steps {
                sh 'mvn clean compile'
            }
        }
        stage('Unit Test'){
        steps {
                sh 'mvn test'
            }
        }
	stage('Integration Test'){
        steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Code Analysis With CheckStyle'){
        steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }
        stage("OWASP Dependency Check"){
        steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.html'
            }
        }
        stage('Trivy FS Check') {
        steps {
                sh "trivy fs ."
            }
        }
        stage('SonarQube Analysis') {
            environment {
             SCANNER_HOME = tool 'sonar-scanner'
          }
        steps{
                withSonarQubeEnv('sonarserver') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic '''

                }
            }
        }
        stage('Git Push to sc-staging') {
	  environment {
                commitmsg = "'CodeChange'"
            }
        steps {
	       script{
                   withCredentials([
                    gitUsernamePassword(credentialsId: 'mygitid', gitToolName: 'Default')
                    ] ) {
                    sh '''
                    git add .
		    git remote -v
                    git status
		    git commit -a -m ${commitmsg}
                    git branch -M sc-staging
                    git push -u origin sc-staging --force
                    '''
                  } 
	       }
            }
        }
    } 
}
