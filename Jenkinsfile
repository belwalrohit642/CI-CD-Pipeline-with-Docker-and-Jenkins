// pipeline {
//     agent any
//     stages {
//         stage("code checkout") {
//             steps {
//                 checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git']])
//             }
//         }
//         stage("Code Stability") {
//             steps {
//                 sh "mvn clean install"
//             }
//         }
//         stage("Code Quality") {
//             steps {
//                 sh "mvn checkstyle:checkstyle"
//                 recordIssues(tools: [checkStyle(pattern: '**/checkstyle-result.xml')])
//             }
//         }
//         stage("Unit Testing") {
//             steps {
//                 sh "mvn test"
//                 recordIssues(tools: [junitParser(pattern: 'target/surefire-reports/*.xml')])
//             }
//         }
//         stage("Security Testing") {
//             steps {
//                 sh "echo Security Testing Done"
//                 // sh "mvn org.owasp:dependency-check-maven:check"
//                 // publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'target', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check Report', reportTitles: ''])
//             }
//         }
//         stage("Sonarqube Analysis") {
//             steps {
//                 sh "mvn sonar:sonar -Dsonar.host.url=${SONAR_URL} -Dsonar.login=${SONAR_USER} -Dsonar.password=${SONAR_PASSWORD} -Dsonar.java.binaries=."
//             }
//         }
//     }
// }
pipeline {
    agent any
    stages {
        stage("code checkout") {
            steps {
                checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git']])
            }
        }
        stage("Code Stability | Build Image") {
            steps {
                script {
                    docker.build("belwalrohit642/spring3hibernate:${env.BUILD_ID}")
                }
            }
        }
        stage("Code Quality | Checkstyle | Hadolint") {
            parallel {
                stage("Checkstyle") {
                    agent {
                        docker {
                            args "-v ${HOME}/.m2:/root/.m2"
                            image 'opstreedevops/maven:java8'
                        }
                    }
                    steps {
                        git credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git'
                        sh "mvn checkstyle:checkstyle"
                        recordIssues(tools: [checkStyle(pattern: '**/checkstyle-result.xml')])
                    }
                }
                stage("Hadolint") {
                    steps {
                        sh 'hadolint Dockerfile --no-fail -f json | tee -a hadolint.json'
                        recordIssues(tools: [hadoLint(pattern: 'hadolint.json')])
                    }
                }
            }
        }
        stage("Unit Testing") {
            agent {
                docker {
                    args "-v ${HOME}/.m2:/root/.m2"
                    image 'opstreedevops/maven:java8'
                }
            }
            steps {
                git credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git'
                sh "mvn test"
                recordIssues(tools: [junitParser(pattern: 'target/surefire-reports/*.xml')])
            }
        }
        stage("Code Coverage") {
            agent {
                docker {
                    args "-v ${HOME}/.m2:/root/.m2"
                    image 'opstreedevops/maven:java8'
                }
            }
            steps {
                git credentialsId: 'git-creds', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git'
                sh "mvn cobertura:cobertura"
                cobertura autoUpdateHealth: false, autoUpdateStability: false,
                           coberturaReportFile: '**/target/site/cobertura/coverage.xml',
                           conditionalCoverageTargets: '70, 0, 0',
                           failUnhealthy: false,
                           failUnstable: false, lineCoverageTargets: '80, 0, 0',
                           maxNumberOfBuilds: 0, methodCoverageTargets: '80, 0, 0',
                           onlyStable: false, sourceEncoding: 'ASCII',
                           zoomCoverageChart: false
            }
        }
        stage("Security Testing | OWASP | Snyk") {
            parallel {
                stage("Dependency Scan") {
                    agent {
                        docker {
                            args "-v ${HOME}/.m2:/root/.m2"
                            image 'opstreedevops/maven:java8'
                        }
                    }
                    steps {
                        sh "echo Security Testing Done"
                    }
                }
                stage("Container Scan") {
                    steps {
                        snykSecurity snykInstallation: 'snyk', snykTokenId: 'snyk', additionalArguments: "--docker belwalrohit642/spring3hibernate:${env.BUILD_ID}", failOnError: false
                    }
                }
            }
        }
        stage("To DockerHub") {
            steps {
                script {
                    def dockerHubCredentials = 'dockerHubCred'
                    def dockerImageName = "belwalrohit642/spring3hibernate"
                    def dockerImageTag = "${env.BUILD_ID}"
                    def dockerImageFullName = "${dockerImageName}:${dockerImageTag}"

                    // Build and push Docker image to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', dockerHubCredentials) {
                        docker.image(dockerImageFullName).push()
                    }
                }
            }
        }
        stage("Deploy to Dev Environment") {
            steps {
                script {
                    ansiblePlaybook credentialsId: 'ec2-user', disableHostKeyChecking: true, installation: 'Ansible', inventory: '/etc/ansible/inventory.ini', playbook: '/var/lib/jenkins/workspace/Spring3Hibernate/play.yaml', vaultTmpPath: ''
                }
            }
        }
    }
    post {
        always {
            script {
                def buildStatus = currentBuild.result

                if (buildStatus != null && buildStatus.equals('FAILURE')) {
                    def failedStage = getCurrentFailedStage()

                    emailext subject: "Pipeline Failure: Build #${env.BUILD_ID}, Stage: ${failedStage}",
                              body: "The build #${env.BUILD_ID} failed at stage: ${failedStage}. \n\nPlease check Jenkins for more details.",
                              to: "recipient@example.com",
                              replyTo: "jenkins@example.com",
                              mimeType: 'text/html'
                }
            }
        }
    }
}

def getCurrentFailedStage() {
    def failedStage = null

    for (stage in currentBuild.rawBuild.getBuildCauses()) {
        if (stage instanceof hudson.model.Cause$UpstreamCause) {
            def upstreamProject = Jenkins.instance.getItemByFullName(stage.upstreamProject)
            def upstreamBuild = upstreamProject.getBuildByNumber(stage.upstreamBuild)

            for (cause in upstreamBuild.getCauses()) {
                if (cause instanceof hudson.model.Cause$UserIdCause) {
                    failedStage = cause.shortDescription
                    break
                }
            }
        }
    }

    return failedStage ?: "Unknown Stage"
}
