pipeline {
    agent any
    environment {
        MODE_ENV = 'test'
        VERCEL_TOKEN = credentials('vercel-token')
    }

    options {
        skipDefaultCheckout(true) // skip the default checkout
    }


    stages {

        stage('clean up code ') {
            steps {
                cleanWs()
            }
        }

        stage('checkout using SCM') {
            steps{
                checkout scm   // checkout the code 
            }
        }

        stage('Take approval'){
            steps {
                timeout(time: 1, unit: 'MINUTES'){
                    input message: 'Do you want to proceed?', ok: 'Proceed'
                }
                
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u root'
                    reuseNode true // Reuse the node for the next stages
                }
            }
            steps {
                script {
                    // Show the initial state of files
                    sh 'ls -l'
                    
                    // Show Node.js and npm versions
                    sh 'node --version'
                    sh 'npm --version'
                    
                    // Install dependencies and run build
                    // sh 'chown -R 992:992 "/ .npm"'

                    sh 'npm install'
                    sh 'npm run build'
                    
                    // Show the final state of files after build
                    sh 'ls -l'
                }
            }
        }

        stage('Test'){
            agent {
                docker {
                    image 'node:22.11.0-alpine3.20'
                    args '-u root'
                    reuseNode true // Reuse the node for the next test
                }
            }
            steps {
                sh '''
                    npm run test 
                    test -f dist/index.html
                '''
            }
        }

        stage("Deploy"){
            agent {
                docker {
                    image 'node:22.11.0-alpine3.20'
                    args '-u root'
                    reuseNode true // Reuse the node for the next test
                }
            }

            steps{
                sh '''
                    npm install -g vercel
                    echo $MODE_ENV
                    vercel --prod --token=$VERCEL_TOKEN --confirm --name=cicdproject
                '''
            }
        }
    }
}
