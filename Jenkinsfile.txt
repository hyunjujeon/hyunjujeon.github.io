pipeline {
    agent { label 'slave01' }

    tools { 
        maven 'Maven3.9.6'
        jdk "JDK1.8"
    }

    environment {
        JAVA_HOME = '/var/jenkins_home/openjdk-1.8.0.402.b06-1'
    }
    
    
    
    stages {
        
        stage('github-clone') {
            steps {
                git branch: 'main', credentialsId: 'github_key', url: 'https://github.com/ddaja121/study.git'
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }

        
        // Pulish over SSH
        stage('Deploy') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: '46_study_board', // 위에서 만든 SSH 의 Name 을 적을것
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: '/home/tomcat/jenkins/was_restart.sh',
                                    execTimeout: 120000, // 쉼표 추가
                                    flatten: false,
                                    makeEmptyDirs: false,
                                    noDefaultExcludes: false,
                                    patternSeparator: '[, ]+',
                                    remoteDirectory: '',
                                    removePrefix: 'target',
                                    sourceFiles: 'target/*.war'
                                )
                            ],
                            verbose: true
                        )
                    ]
                )
            }
        }

            
   		// stage...
   	}
}
