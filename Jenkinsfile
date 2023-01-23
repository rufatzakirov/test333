pipeline {
    agent any
    environment {
	ANSIBLE_PRIVATE_KEY=credentials('ssh-key')
	}
    triggers {
      pollSCM '* * * * *'
	}
    stages {
        stage("TEST") {
            steps {
                echo "--- Tests ---"
            }
        }
        stage("Build") {
            steps {
                echo "--- Build ---"
                sh "docker build -t rufatzakirov/$JOB_NAME:v$BUILD_ID ."
            }
        }
        stage("Push DockerHUB") {
            steps {
                echo "--- Push DockerHUB ---"
                withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'pass', usernameVariable: 'user')]) {
                sh "docker login -u $user -p $pass"
                sh "docker push rufatzakirov/$JOB_NAME:v$BUILD_ID"
                }
            }
        }
        stage("Deploy") {
            steps {
		withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'password', usernameVariable: 'username')]) {
                echo "--- Deploy ---"
                sh "ansible-playbook playbook.yaml -i inventory --private-key=$ANSIBLE_PRIVATE_KEY -u ansible --become -e username=$username -e password=$password -e BUILD_ID=$BUILD_ID "
		}
            }
        }
    }
    post {
     success { 
        withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
        sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : OK *Published* = YES'
        """)
        }
     }

     aborted {
        withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
        sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC *Branch*: ${env.GIT_BRANCH} *Build* : `Aborted` *Published* = `Aborted`'
        """)
        }
     
     }
     failure {
        withCredentials([string(credentialsId: 'botSecret', variable: 'TOKEN'), string(credentialsId: 'chatId', variable: 'CHAT_ID')]) {
        sh  ("""
            curl -s -X POST https://api.telegram.org/bot${TOKEN}/sendMessage -d chat_id=${CHAT_ID} -d parse_mode=markdown -d text='*${env.JOB_NAME}* : POC  *Branch*: ${env.GIT_BRANCH} *Build* : `not OK` *Published* = `no`'
        """)
        }
     }

 }

}
