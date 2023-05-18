pipeline {
    agent any

    stages {
        stage("Clone Git Repository") {
            steps {
                echo "Cloning Git Repository"
                git branch: 'BRANCH_NAME', credentialsId: "CREDENTIAL_ID", url: "git@REPO_URL"
                sh "git config user.name 'USERNAME'"
                sh "git config user.email 'USER_EMAIL'"
            }
        }
        
        stage("Remove Previous Resource Quota File") {
            when { expression { return fileExists ("resource_quota.yaml") } }
            steps {
                echo "File Exists"
                echo "Removing: resource_quota.yaml"
                sh "rm -f resource_quota.yaml"
            }
        }
        
        stage("Create New Resource Quota File") {
            steps {
                script {
                    // Create config variable
                    def config = readYaml file: "template_resource_quota.yaml"

                    // Update content from template; config variable
                    config.metadata.namespace = params.namespace
                    config.metadata.labels."quota-tier" = params.tier
                    config.metadata.annotations."kubernetes.io/quota-tier" = params.tier
                    config.spec.hard.cpu = params.cpu
                    config.spec.hard.memory = params.memory
                    config.spec.hard."limits.cpu" = params.burstcpu
                    config.spec.hard."limits.memory" = params.burstmem
                    
                    // Write config variable updates to resource_quota.yaml file
                    writeYaml file: "resource_quota.yaml", data: config

                    // This isn't really necessary, just left it in for now
                    def read = readYaml file: "resource_quota.yaml"
                }
                
                // Print out template
                sh "cat template_resource_quota.yaml"
                // Print out resource_quota.yaml
                sh "cat resource_quota.yaml"
                
                // Add updated files to git repository and commit
                sh "git add *"
                sh "git commit -m 'Updated with Jenkins Pipeline Job - ${JOB_NAME}: ${BUILD_NUMBER}'"
            }
        }
        
        stage("Push to Git Repository"){
            steps{
                withCredentials([sshUserPrivateKey(credentialsId: "CREDENTIAL_ID", keyFileVariable: "Default")]) {
                    // Push updates to git repository
                    sh "git push -u origin main"
                }
            }
        }
    
    }
    post {
        always {
            deleteDir()
        }
    }
}