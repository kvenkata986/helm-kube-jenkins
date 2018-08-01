pipeline {
    // Clean  workspace before doing anything
    agent any
    stages {
        stage ('Clone') {
            steps {
            checkout scm
            }
        }
        stage ('Build') {
            steps {
                git url: 'https://github.com/kvenkata986/helm-kube-jenkins.git'
            }
        }
        stage ('Tests') {
            environment {
                CHARTNAME = 'helm-kube-jenkins'
            }
            steps {
              parallel 'Syntax Checks': {
                  sh "echo 'shell scripts to run static tests...'"
              },
              'Security Scans': {
                  sh "echo ' $CHARTNAME shell scripts to run Security Scans' "
              },
              'Helm Linting': {
                  sh '''
                    CHARTNAME=`grep name Chart.yaml  | awk '{print $2}'`
                    mkdir -p /tmp/$CHARTNAME
                    sudo cp -Rp . /tmp/$CHARTNAME
                    cd /tmp/$CHARTNAME
                    sudo -H -u hekujen bash -c 'helm lint .'
                    rm -rf /tmp/$CHARTNAME
                  '''
              }
            }
        }
        stage ('Deploy in Testing Environment') {
            steps {
              sh '''
                sudo -H -u hekujen bash -c 'helm upgrade --install --namespace testing helm-testing . '
              '''
              timeout(time: 1, unit: 'HOURS') {
                 input message: "Does Pre-Production look good?"
              }
            }
        }
        stage ('Deploy in Prod Environment') {
            steps {
              sh '''
                sudo -H -u hekujen bash -c 'helm upgrade --install --namespace production helm-prod . '
              '''
            }
        }
    }
}
