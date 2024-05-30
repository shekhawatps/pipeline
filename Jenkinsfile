pipeline {
    agent any

    parameters {
        string(name: 'GCP_PROJECT', defaultValue: 'your-gcp-project-id', description: 'GCP Project ID')
        string(name: 'VM_NAMES', defaultValue: 'vm1,vm2,vm3', description: 'Comma-separated list of GCP VM names to manage')
        choice(name: 'ACTION', choices: ['start', 'stop'], description: 'Action to perform on the VMs')
    }

    environment {
        GOOGLE_APPLICATION_CREDENTIALS = credentials('your-jenkins-credential-id')
    }

    stages {
        stage('Setup') {
            steps {
                script {
                    // Install Google Cloud SDK if not already installed
                    if (!fileExists('/usr/bin/gcloud')) {
                        sh '''
                            curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-367.0.0-linux-x86_64.tar.gz
                            tar -zxvf google-cloud-sdk-367.0.0-linux-x86_64.tar.gz
                            ./google-cloud-sdk/install.sh -q
                            export PATH=$PATH:/usr/bin/google-cloud-sdk/bin
                        '''
                    }
                    
                    // Authenticate with Google Cloud
                    sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
                    sh "gcloud config set project ${params.GCP_PROJECT}"
                }
            }
        }

        stage('Approval') {
            steps {
                script {
                    def userInput = input(
                        id: 'Approval', message: 'Approve to proceed with VM operation?', 
                        parameters: [
                            choice(name: 'ACTION', choices: ['start', 'stop'], description: 'Action to perform on the VMs'),
                            string(name: 'VM_NAMES', defaultValue: params.VM_NAMES, description: 'Comma-separated list of GCP VM names to manage')
                        ]
                    )
                    env.ACTION = userInput.ACTION
                    env.VM_NAMES = userInput.VM_NAMES
                }
            }
        }

        stage('Perform Action') {
            steps {
                script {
                    def vmNames = env.VM_NAMES.split(',').collect { it.trim() }
                    for (vmName in vmNames) {
                        if (env.ACTION == 'start') {
                            sh "gcloud compute instances start ${vmName}"
                        } else if (env.ACTION == 'stop') {
                            sh "gcloud compute instances stop ${vmName}"
                        } else {
                            error "Unsupported action: ${env.ACTION}"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // Optionally handle cleanup or final steps
                if (env.ACTION == 'start') {
                    echo "VM(s) were started as part of this job."
                } else if (env.ACTION == 'stop') {
                    echo "VM(s) were stopped as part of this job."
                }
            }
        }
    }
}
