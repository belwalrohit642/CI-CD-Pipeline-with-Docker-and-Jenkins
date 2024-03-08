pipeline{
    agent any
    stages {
stage("code checkout"){
    steps{
checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'gitCred', url: 'https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git']])
}
}
}
}
