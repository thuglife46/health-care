node {
    def mavenHome
    def mavenCMD
    def docker
    def dockerCMD
    def tagName
    
    stage('prepare environment'){
        echo 'Initialize the variables'
        mavenHome = tool name: 'myMaven' , type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        docker = tool name: 'myDocker' , type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD = "${docker}/bin/docker"
        tagName = "1.0"
    }
    stage ('code checkout'){
        try{
        echo 'pulling the code from github repo'
        git 'https://github.com/niladrimondal/star-agile-health-care.git'
        }
        catch(Exception e){
            echo 'Exception Occur'
            currentBuild.result = "FAILURE"
            emailext body: '''Hello Staragile Deveops

            The Build Number ${BUILD_NUMBER} is Failed. Please look into that.

            Thanks,''', subject: 'The jenkis Job ${JOB_NAME} is Failed ', to: 'niladrimondal.mondal@gmail.com'
        }
    }
    stage('Build the application'){
        echo 'clean and compile and test package'
        //sh 'mvn clean package'
        sh "${mavenCMD} clean package"
    }
    stage('publish html reports'){
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/StrarAgileDevopsPipeline/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report Staragile/var/lib/jenkins/workspace/StrarAgileDevopsPipeline', reportTitles: '', useWrapperFileDirectly: true])
    }
    stage('Build the DockerImage of the application'){
        try{
        echo 'creating the docker image'
		// if you get permission denied issue
        //sudo usermod -a -G docker jenkins
        //restart Jenkins
        //or add sudoers file below line
        //jenkins ALL=(ALL) NOPASSWD:ALL
        sh "${dockerCMD} build -t niladrimondaldcr/medicure:${tagName} ."
        
        }
        catch(Exception e){
            echo 'Exception Occur'
            currentBuild.result = "FAILURE"
            emailext body: '''Hello Staragile Deveops

            The Build Number ${BUILD_NUMBER} is Failed. Please look into that.

            Thanks,''', subject: 'The jenkis Job ${JOB_NAME} is Failed ', to: 'niladrimondal.mondal@gmail.com'
            
        }
    }
    stage('push the docker image to dockerhub'){
        echo 'pushing docker image'
        withCredentials([string(credentialsId: 'docker-password', variable: 'DockerPassword')]) {
        // some block
        sh "${dockerCMD} login -u niladrimondaldcr -p ${DockerPassword}"
        sh "${dockerCMD} push niladrimondaldcr/medicure:${tagName}"
        }
    }
    stage('deploy the application'){
        
        ansiblePlaybook become: true, credentialsId: 'ansiblekey', disableHostKeyChecking: true, installation: 'MyAnsible', inventory: '/etc/ansible/hosts', playbook: 'ansible-playbook.yml'
    }
}
