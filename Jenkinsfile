node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('prepare environment'){
        echo 'initialize the variables'
        mavenHome = tool name: 'myMaven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'myDocker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName="3.0"
    }
    stage('code checkout'){
        try{
        echo 'code checkout'
        git 'https://github.com/JaganPothina/Insurance-Project'
        }
        catch(Exception e){
            echo 'exception occur in stage code checkout'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The jenkins job ${JOB_NAME} is failed. please look into the ${BUILD_NUMBER}''', subject: 'job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'jaganpothina@gmail.com'
        }
    }
    stage('Build the application'){
        echo ' clean and compile and test package'
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"
    }
    
    stage('publish test reports'){
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/insureme/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
    }
    
    stage ('containerize the application'){
        try{
        echo 'build the docker image'
        // if you get permission denied issue
        //sudo usermod -a -G docker jenkins
        //restart Jenkins
        //or add sudoers file below line
        //jenkins ALL=(ALL) NOPASSWD:ALL
        sh "${dockerCMD} build -t 9182924985/insureme:${tagName} ."
        }
        catch(Exception e){
            echo 'exception occur in stage code checkout'
            currentBuild.result = "FAILURE"
            emailext body: '''Dear All,
            The jenkins job ${JOB_NAME} is failed. please look into the ${BUILD_NUMBER}''', subject: 'job ${JOB_NAME} ${BUILD_NUMBER} is failed', to: 'jaganpothina@gmail.com'
        }
    }
    stage ('push docker image to dockerhub')
    echo 'pushing the docker image to DockerHub'
    withCredentials([string(credentialsId: 'docker-password', variable: 'DockerPassword')]) {
    sh "${dockerCMD} login -u 9182924985 -p ${DockerPassword}"
    sh "${dockerCMD} push 9182924985/insureme:${tagName}"
    }
    stage ('Configure and Deploy to the test-server'){
        ansiblePlaybook become: true, credentialsId: 'ansiblekey', disableHostKeyChecking: true, installation: 'myAnsible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
    
}
