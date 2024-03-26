pipeline {
    agent any 
    parameters {
        string(name: 'DB_NAME', defaultValue: 'crmwebsite', description: 'Database name')
        string(name: 'DB_PORT', defaultValue: '5432', description: 'Database port')
        string(name: 'SONAR_URL', defaultValue: 'http://20.151.87.193:9000', description: 'Sonarqube URL')
        // Add more parameters as needed
    }
    
    options {
        disableConcurrentBuilds() // Disable concurrent builds to ensure sequential execution
    }
    
    stages {
        stage('Install Ansible') {
            steps {
                script {
                    // Install Ansible using apt-get on Ubuntu
                    sh 'sudo apt-get update -y'
                    sh 'sudo apt-get install software-properties-common -y'
                    sh 'sudo apt-add-repository --yes --update ppa:ansible/ansible'
                    sh 'sudo apt-get install ansible -y'
                    
                    // Check Ansible version
                    sh 'ansible --version'
                }
            }
        }

        stage('Install Python and Pip') {
            steps {
                ansiblePlaybook(
                    playbook: 'required-installations.yaml',
                    inventory: 'localhost',
                    installation: 'ansible'
                )
            }
        } 
        
        stage('Setup Environment') {
            steps {
                script {
                    // Create a virtual environment and activate it
                    sh 'python3 -m venv venv'
                    sh '. venv/bin/activate'
                    // Install dependencies
                    sh 'pip install -r requirements.txt'
                }
            }
        }
        
        stage('Create Database') {
            steps {
                script {
                    // Inject credentials into environment variables
                    withCredentials([
                        string(credentialsId: 'DB_HOST', variable: 'DB_HOST'),
                        string(credentialsId: 'DB_USER', variable: 'DB_USER'),
                        string(credentialsId: 'DB_PASSWORD', variable: 'DB_PASSWORD')
                    ]) {
                        // Set other environment variables
                        env.DB_NAME = "${params.DB_NAME}"
                        env.DB_PORT = "${params.DB_PORT}"

                        // Check if the database exists by running the check_db.sh script
                        def databaseExists = sh(script: 'chmod +x ./check_db.sh && ./check_db.sh', returnStatus: true)
                        if (databaseExists == 0) {
                            echo 'Database already exists. Skipping creation.'
                        } else {
                            // If the database does not exist, run the mydb.py script
                            sh 'python3 mydb.py'
                        }
                    }
                }
            }
        }
        
        stage('Database Migration') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: 'DB_HOST', variable: 'DB_HOST'),
                        string(credentialsId: 'DB_USER', variable: 'DB_USER'),
                        string(credentialsId: 'DB_PASSWORD', variable: 'DB_PASSWORD')
                    ]) {
                        // Set other environment variables
                        env.DB_NAME = "${params.DB_NAME}"
                        env.DB_PORT = "${params.DB_PORT}"
                        
                        // Check if migrations are needed
                        def migrationsNeeded = sh(script: 'python3 manage.py showmigrations --plan', returnStdout: true).trim()
                        if (migrationsNeeded.contains(' (no migrations)')) {
                            echo 'No new migrations found.'
                        } else {
                            // Run migrations
                            sh 'python3 manage.py migrate'
                        }
                    }
                }
            }
        }
         
        stage('Unit testing') {
            steps {
                script {

                        sh 'python3 manage.py test website'
                    }
                }
            }
        }


        /*stage('Static Code Analysis using SonarQube') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqubeToken', variable: 'sonarqubeTokenvariable')]) {
                        // Assuming 'SONAR_URL' is a parameter defined in your pipeline
                        env.SONAR_URL = http://20.151.87.193:9000
                        sh 'sonar-sonar -Dsonar.host.url=${SONAR_URL} -Dsonar.login=$sonarqubeTokenvariable'
                    }
                }
            }
        }
        stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Django-Project"
            GIT_USER_NAME = "mayank91091"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "mayankthukral1810@gmail.com"
                    git config user.name "mayank91091"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" ./deployment.yml
                    git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }*/
    
    
    post {
        success {
            // Send notification on success
            echo 'Pipeline succeeded!'
        }
        failure {
            // Send notification on failure
            echo 'Pipeline failed!'
        }
    }
}
