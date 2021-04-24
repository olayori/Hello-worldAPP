pipeline {
    agent any
    
    stages {
        stage('Checkout SCM') {
            steps {
                checkout changelog: false, poll: false, scm: [$class: 'GitSCM', branches: [[name: '*/kubernetes']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/olayori/Hello-worldAPP.git']]]
            }
        }
        stage('Build Job') {
            tools {
                maven 'Maven 3.8.1'
            }
            steps {
                sh 'mvn -f pom.xml clean install package'
            }
        }
        stage('Deploy to Tomcat Host Server') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'deployer', path: '', url: 'http://18.207.198.232:8090')], contextPath: null, onFailure: false, war: '**/*.war'
            }
        }        
        stage('Ansible Deploy to Docker Host') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'controller-ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /root/jenkins-ansible;
ansible-playbook create-docker-image-container.yml;''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        stage('Ansible push Image DockerHub') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'controller-ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /root/jenkins-ansible;
ansible-playbook push-simple-devops-image.yml;''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: 'webapp/target', sourceFiles: 'webapp/target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
        stage('Ansible deloy to K8s') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'controller-ansible', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '''cd /root/ansible-k8s;
ansible-playbook -i hosts kubernetes-owolade-deployment.yml;
ansible-playbook -i hosts kubernetes-owolade-service.yml;''', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            }
        }
    }    
    post{
        success {
                archiveArtifacts artifacts: 'webapp/target/*.war'
        }    
    }
}
