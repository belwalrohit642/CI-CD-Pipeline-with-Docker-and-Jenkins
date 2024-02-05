pipeline{
    agent any
    stages {
stage("code checkout"){
    steps{
git credentialsId: ‘gitcred’, url:
‘https://github.com/belwalrohit642/CI-CD-Pipeline-with-Docker-and-Jenkins.git’
}
}
        stage("Code Stability | build images"){
            steps{
script {
docker.build(“:${env.BUILD_ID}”)
}
}
}
}
