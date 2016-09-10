
webapp_usrname = ""
webapp_server_url = ""

def buildProject(){
    git branch: 'azure-pipeline', poll: true, url: 'https://github.com/sysgain/game-of-life'     
    sh "mvn clean package" 
} 

def deployProject(){     
    sh "scp ./gameoflife-web/target/gameoflife.war $webapp_usrname@$webapp_server_url:/home/$webapp_usrname/stack/apache-tomcat/webapps/" 
} 

node{     
    stage "Build"     
    buildProject()         
    stage "Deploy to Azure"     
    deployProject() 
    
    sh "pwd"
}
