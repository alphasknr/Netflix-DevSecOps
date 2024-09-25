pipeline {
    agent any
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/alphasknr/Netflix-DevSecOps.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix'''
                }
            }
        }
       stage('OWASP FS Scanning') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'owasp-dependency-check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scanning') {
            steps {
                script {
                    try {
                        sh 'trivy fs . > trivyfs.txt' 
                    }catch(Exception e){
                        input(message: 'Are you sure to proceed?', ok: 'Proceed')
                    }
                }
            }
        }
        stage('Docker Image Build') {
            steps {
                script {
                    sh 'docker system prune -f'
                    sh 'docker container prune -f'
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                        try {
                            def previousBuildNumber = BUILD_NUMBER.toInteger() - 1
                            sh "docker rmi ${Docker_Name}/netflix_clone:${previousBuildNumber} || echo 'No previous image to delete'"
                        } catch (Exception e) {
                            echo 'No previous images are there to delete, or an error occurred during deletion'
                        }
                        sh 'docker build --build-arg API_KEY=2af0904de8242d48e8527eeedc3e19d9 -t ${Docker_Name}/netflix_clone:${BUILD_NUMBER} .'
                    }
                }
            }
        }
        
        stage('Push to Dockerhub') {
            steps {
		        script {
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                    sh 'docker login -u ${Docker_Name} -p ${Docker_Password}'
                    sh 'docker push ${Docker_Name}/netflix_clone:${BUILD_NUMBER}'
                    }
		        }
            }
        }
        stage('Trivy Image Scanning'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                    try{
                        sh 'echo ${Docker_Name}/netflix_clone:${BUILD_NUMBER}'
                        sh 'trivy image ${Docker_Name}/netflix_clone:${BUILD_NUMBER} > trivyimage.txt'
                    }
                    catch(Exception e) {
                        input(message: 'Are you sure to proceed?', ok: 'Proceed')
                    }
                    }
                }
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "Netflix-DevSecOps"
            }
            steps {
                withCredentials([string(credentialsId: 'github-token-id', variable: 'GITHUB_TOKEN'),
                                 string(credentialsId: 'github-mail-id', variable: 'GIT_EMAIL'),
                                 string(credentialsId: 'github-username-id', variable: 'GIT_USERNAME')]) {
                    sh """
                        # Set Git configuration with user credentials
                        git config user.email "$GIT_EMAIL"
                        git config user.name "$GIT_USERNAME"
                        
                        # Update the deployment YAML file using sed
                        sed -i.bak "s/netflix-clone:.*/netflix-clone:${BUILD_NUMBER}/g" ./Manifest_Files/deployment.yaml
                        
                        # Add and commit changes
                        git add ./Manifest_Files/deployment.yaml
                        git commit -m "Update deployment Image to version ${BUILD_NUMBER}"
        
                        # Push changes to GitHub (without exposing the token in logs)
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USERNAME}/${GIT_REPO_NAME}.git HEAD:main
                    """
                }
            }
        }
    }
}
