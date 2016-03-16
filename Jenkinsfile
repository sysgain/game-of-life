#!groovy

properties([
   [$class: 'BuildDiscarderProperty',
      strategy: [$class: 'LogRotator', numToKeepStr: '10', artifactNumToKeepStr: '10']
   ]
])

docker.image('cloudbees/java-build-tools:0.0.7.1').inside {

    checkout scm

    def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

    stage 'Build'
    wrap([$class: 'ConfigFileBuildWrapper',
        managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

        sh "mvn -s ${mavenSettingsFile} clean source:jar javadoc:javadoc checkstyle:checkstyle pmd:pmd findbugs:findbugs package"

        step([$class: 'ArtifactArchiver', artifacts: 'target/*.war'])
        step([$class: 'WarningsPublisher', consoleParsers: [[parserName: 'Maven']]])
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
        step([$class: 'JavadocArchiver', javadocDir: 'gameoflife-core/target/site/apidocs/'])

        // Use fully qualified hudson.plugins.checkstyle.CheckStylePublisher if JSLint Publisher Plugin or JSHint Publisher Plugin is installed
        step([$class: 'hudson.plugins.checkstyle.CheckStylePublisher', pattern: '**/target/checkstyle-result.xml'])
        // In real life, PMD and Findbugs are unlikely to be used simultaneously
        step([$class: 'PmdPublisher', pattern: '**/target/pmd.xml'])
        step([$class: 'FindBugsPublisher', pattern: '**/findbugsXml.xml'])
        step([$class: 'AnalysisPublisher'])
    }
    stash name: 'acceptance-tests', includes: 'gameoflife-acceptance-tests/,target/ROOT.war'
}

stage name:'Deploy to OpenShift', concurrency: 1
mail \
    to: 'alobato@cloudbees.com',
    subject: "Deploy version #${env.BUILD_NUMBER} on http://game-of-life-qa.openshift.beesshoop.org/ ?",
    body: """\
       Deploy game-of-life#${env.BUILD_NUMBER} and start web browser tests on http://game-of-life-qa.openshift.beesshoop.org/ ?
       Approve/reject on ${env.BUILD_URL}.
       """

input "Deploy on http://game-of-life-qa.openshift.beesshop.org/ and run Selenium tests?"
checkpoint 'Deploy to QA'

docker.image('cloudbees/java-build-tools:0.0.7.1').inside {
    //Requires CloudBees OpenShift CLI Plugin to
    //automatically install OC client and log in to the server
    wrap([$class: 'OpenShiftBuildWrapper', 
            url: 'https://openshift.beesshop.org:8443',
            credentialsId: 'openshift-admin-aws',
            insecure: true, //Don't check server certificate
            ]) {

            // oc & source2image
            sh """
            oc project game-of-life
            oc start-build j2ee-application-build --wait=true
            oc deploy frontend --latest > .openshift-deploy-number
            """
            def deployMessage = readFile(".openshift-deploy-number")
            def deployNumber = deployMessage.substring(deployMessage.indexOf('#')).trim()
            echo "$deployMessage".trim()

            // Wait for OpenShift deployment
            timeout(time: 5, unit: 'MINUTES') {
                waitUntil {
                    sleep 5L //poll only every 5 seconds
                    sh "oc deploy frontend > .openshift-build-status"
                    // parse `describe-environment-health` output
                    def openshiftDeployStatus = readFile(".openshift-build-status")
                    echo "Checking: '$deployNumber deployed'"
                    def isDeployed = openshiftDeployStatus.indexOf("$deployNumber deployed")
                    echo "$openshiftDeployStatus"
                    echo "$isDeployed"
                    return isDeployed>0
                }
            }


    }
}

stage name: 'Test with Selenium', concurrency: 1

node {
    unstash 'acceptance-tests'

    // web browser tests are fragile, test up to 3 times
    retry(3) {
        docker.image('cloudbees/java-build-tools:0.0.6').inside {
            def mavenSettingsFile = "${pwd()}/.m2/settings.xml"

            wrap([$class: 'ConfigFileBuildWrapper',
                managedFiles: [[fileId: 'maven-settings-for-gameoflife', targetLocation: "${mavenSettingsFile}"]]]) {

                sh """\
                   # debug info
                   # curl http://game-of-life-qa.openshift.beesshop.org/
                   # curl -v http://localhost:4444/wd/hub

                   cd gameoflife-acceptance-tests
                   mvn -B -V -s ${mavenSettingsFile} verify -Dwebdriver.driver=remote -Dwebdriver.remote.url=http://localhost:4444/wd/hub -Dwebdriver.base.url=http://game-of-life-qa.openshift.beesshop.org/
                """
            }
        }
    }
}