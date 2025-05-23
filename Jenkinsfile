pipeline {
    agent any
    environment {
        SONAR_SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clone Repostitory') {
            steps {
                git branch: 'main', url: 'https://github.com/gujjar-aditya/project-insurance.git'
            }
        }
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }
        stage('Nexus artifact upload') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'insure-me', classifier: '', file: 'target/insure-me-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'nexus-cred', groupId: 'com.project.staragile', nexusUrl: '13.201.82.148:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '0.0.1-SNAPSHOT'
            }
        }
        stage('Docker Build') {
            steps {
                echo 'Building image...'
                sh 'docker build --no-cache -t adityagujjar/project-insurance:V1.0 .'
                sh 'docker images'
            }   
        }       
        stage('Docker hub login and push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push adityagujjar/project-insurance:V1.0'
                }    
            }
        }
        stage('Deploy using ansible') {
            steps {
                ansiblePlaybook credentialsId: 'ansible-ssh', disableHostKeyChecking: true, installation: 'ansible2', inventory: 'inventory.ini', playbook: 'deployment_playbook.yml', vaultTmpPath: ''
            }
        }
    }
}




