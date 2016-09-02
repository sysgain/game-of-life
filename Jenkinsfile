def local_path="gameoflife-web/target"
def war="gameoflife.war"
def target="/home/cb-jenkins/stack/apache-tomcat/webapps"

def ensureMaven() {
    env.PATH = "${tool 'maven-3.3'}/bin:${env.PATH}"
}

def buildProject(){
    //ensureMaven()
    git branch: 'azure-pipeline', poll: true, url: 'https://github.com/sysgain/game-of-life'
    sh "mvn clean package"
}

def deployProject(){
    
    withCredentials([[$class: 'UsernamePasswordMultiBinding', 
        credentialsId: 'azure-deployment-id', 
        passwordVariable: '_password', 
        usernameVariable: '_user']]) {

        sh "curl -T ${local_path}/${war} ftps://\"${env._user}\":${env._password}@${env.azureHost}${target}/"
    }
}

node{
    stage "Build"
    buildProject()
    
    stage "Deploy to Azure"
    //deployProject()
}
