def buildProject(){
    git branch: 'azure-pipeline', poll: true, url: 'https://github.com/sysgain/game-of-life'     
    sh "mvn clean package" 
} 

def deployProject(){     
    sh "scp ./gameoflife-web/target/gameoflife.war cb-jenkins@bitnamidnsnq5gdvwi5sfcy.eastus.cloudapp.azure.com:/home/cb-jenkins/stack/apache-tomcat/webapps/" 
} 

node{     
    stage "Build"     
    buildProject()         
    stage "Deploy to Azure"     
    deployProject() 
    
    sh "pwd"
}
