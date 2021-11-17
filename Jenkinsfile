properties([pipelineTriggers([githubPush()])])

pipeline {
    agent {label 'slave'}
    environment {
        imagename = "vzhyhalau/tmscourseproject"
        registryCredential = 'git'
        dockerImage = ''
        CLASS           = "GitSCM"
        BRANCH          = "main"
        GIT_CREDENTIALS = "TMS_Project"
        GIT_URL         = "git@github.com:vszhigalov/TMS_Course_Project.git"

        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }
    stages {
      stage('Notification on Slack Start') {
            steps {
                slackSend channel: '#test', message: 'Job Start', blocks: [
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "*Run configuration*"
                      ]
                    ],
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "Start Checkout SCM, Prepare build image"
                      ]
                     ]
                    ]
               }
             }
      stage('Checkout SCM') {
            steps {
                checkout([
                    $class: "${CLASS}",
                    branches: [[name: "${BRANCH}"]],
                    userRemoteConfigs: [[
                    url: "${GIT_URL}",
                    credentialsId: "${GIT_CREDENTIALS}",
                    ]]
                ])
            }
        }
       stage("Prepare build image") {
            steps {
              withCredentials([usernamePassword(credentialsId: 'docker-login-creds', passwordVariable: 'password', usernameVariable: 'username')]){
                sh "sudo docker build -f Dockerfile . -t ${username}/tmscourseproject:${BUILD_ID}"
                sh "sudo docker login -u${username} -p${password}"
                sh "sudo docker push ${username}/tmscourseproject:${BUILD_ID}"
              }
            }
        }
       stage('Notification on Slack Comleted build/push image') {
            steps {
                slackSend channel: '#test', message: 'Job processed', blocks: [
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "*Image pushed*"
                      ]
                    ],
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "Image on DockerHub"
                      ]
                     ]
                    ]
               }
             }
       stage('Notification on Slack start ec2.py and run Ansible-playbook') {
            steps {
                slackSend channel: '#test', message: 'Job processed', blocks: [
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "*start ec2.py and run Ansible-playbook*"
                      ]
                    ],
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "Run dynamic inventory, generate file host for ansible playbook and run deploy"
                      ]
                     ]
                    ]
               }
             }
       stage("run ec2.py") {
            steps {
                sh "chmod +x ec2.py"
                sh "pwd"
                sh "./ec2.py --list"
                //sh "ansible -i ec2.py all -m ping"
                //sh "ansible-playbook -i ec2.py tag_Owner_VZhig -m ping -u ubuntu --private-key "
            }
        }
       stage("Ansible") {
            steps {
               ansiblePlaybook become: true, becomeUser: 'root', credentialsId: 'TMS-ireland', disableHostKeyChecking: true, installation: 'Ansible', inventory: 'ec2.py', playbook: 'deploy.yml'
               //sh "sudo /usr/bin/ansible-playbook deploy.yml -i ec2.py -b --become-user root --private-key /home/ubuntu/TMS-ireland.key -u ubuntu"
            }
          }

       stage('Notification on Slack finish Job') {
            steps {
                slackSend channel: '#test', message: 'Job finish', blocks: [
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "*Deploy completed*"
                      ]
                    ],
                    [
                      "type": "section",
                      "text": [
                        "type": "mrkdwn",
                        "text": "Lets drink coffee"
                      ]
                     ]
                    ]
               }
             }
    }
}
