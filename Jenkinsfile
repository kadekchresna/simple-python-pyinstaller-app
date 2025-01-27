pipeline {
    agent any
    options {
        skipStagesAfterUnstable()
    }
    environment {
        EC2_USER = 'ec2-user' 
        EC2_HOST = 'ec2-3-95-180-17.compute-1.amazonaws.com' 
        APP_DIR = '/home/ec2-user/python-simple-app'
        SSH_KEY_ID = 'ssh-aws-chresna.dev'
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'kadekchresna/py-submision:3.9'
                    args '-p 3000:3000'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'kadekchresna/py-submision:3.9'
                    args '-p 3000:3000'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }
        stage('Manual Approval') {
            steps {
                input message: 'Continue to Deploy step?'
            }
        }
        
        stage('Deploy') { 
            agent { 
                label 'build-in' 
            }
            steps {
                sshagent(credentials: [SSH_KEY_ID]) {
                    sh """
                    cd /var/jenkins_home/workspace/python-simple-app
                    tar -czf build.tar.gz -C sources .
                    scp -o StrictHostKeyChecking=no build.tar.gz ${EC2_USER}@${EC2_HOST}:${APP_DIR}
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "cd ${APP_DIR} && tar -xzf build.tar.gz && pyinstaller --onefile add2vals.py && rm -f build.tar.gz && ./dist/add2vals 2 3"
                    """
                }
                sleep(60)
                sshagent(credentials: [SSH_KEY_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} "cd ${APP_DIR} && rm ./dist/add2vals"
                    """
                }
            }
        }
    }
}