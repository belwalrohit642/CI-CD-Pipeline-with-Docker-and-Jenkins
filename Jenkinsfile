pipeline {
    agent any
    stages {
        stage("code checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git']])
            }
        }
        stage("Code Stability") {
            steps {
                sh "mvn clean install"
            }
        }
        stage("Code Quality") {
            steps {
                sh "mvn checkstyle:checkstyle"
                recordIssues(tools: [checkStyle(pattern: '**/checkstyle-result.xml')])
            }
        }
        stage("Unit Testing") {
            steps {
                sh "mvn test"
                recordIssues(tools: [junitParser(pattern: 'target/surefire-reports/*.xml')])
            }
        }
        stage("Security Testing") {
            steps {
                sh "echo Security Testing Done"
                // sh "mvn org.owasp:dependency-check-maven:check"
                // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check Report', reportTitles: ''])
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_URL} -Dsonar.login=${SONAR_USER} -Dsonar.password=${SONAR_PASSWORD} -Dsonar.java.binaries=."
            }
        }
    }
}
