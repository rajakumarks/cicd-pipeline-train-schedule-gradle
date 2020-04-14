pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('DeployToStaging') {
          when { branch 'mater' }
          steps {
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordvariable: 'PASSWORD')]) {
              sshPublisher(
                failOnError: true,
                continueOnError: false,
                publishers: [
                  sshPublisherDesc(
                    configName: 'staging',
                    sshCredentials: [
                      username: "$USERNAME",
                      encryptedPassphrase: "$PASSWORD"
                    ],
                    transfers: [
                      sshTransfer(
                        sourceFiles: 'dist/trainSchedule.zip',
                        removePrefix: 'dist/',
                        remoteDirectory: '/tmp',
                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule' 
                      )
                    ]
                   )
                ]
              )
            }
          }
        }
         stage('DeployToProduction') {
          when { branch 'mater' }
          steps {
              input 'Does the Staging env look OK?'
              milestone(1)
            withCredentials([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordvariable: 'PASSWORD')]) {
              sshPublisher(
                failOnError: true,
                continueOnError: false,
                publishers: [
                  sshPublisherDesc(
                    configName: 'production',
                    sshCredentials: [
                      username: "$USERNAME",
                      encryptedPassphrase: "$PASSWORD"
                    ],
                    transfers: [
                      sshTransfer(
                        sourceFiles: 'dist/trainSchedule.zip',
                        removePrefix: 'dist/',
                        remoteDirectory: '/tmp',
                        execCommand: 'sudo /usr/bin/systemctl stop train-schedule && rm -rf /opt/train-schedule/* && unzip /tmp/trainSchedule.zip -d /opt/train-schedule && sudo /usr/bin/systemctl start train-schedule' 
                      )
                    ]
                   )
                ]
              )
            }
          }
        }
    }
}
