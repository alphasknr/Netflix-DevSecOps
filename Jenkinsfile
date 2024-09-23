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
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'OWASP DP-Check'
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
                    withCredentials([usernamePassword(credentialsId: 'Docker_Cred_ID', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                        sh 'docker build --build-arg API_KEY=2af0904de8242d48e8527eeedc3e19d9 -t ${Docker_Name}/netflix_clone:${BUILD_NUMBER} .'
                    }
                }
            }
        }
        stage('Trivy Image Scanning'){
            steps{
                sh 'trivy image netflix > trivyimage.txt'
                script{
                    withCredentials([usernamePassword(credentialsId: 'Docker_Cred_ID', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                    sh 'trivy image ${Docker_Name}/netflix_clone:${BUILD_NUMBER} > trivyimage.txt'
                    }
                    input(message: 'Are you sure to proceed?', ok: 'Proceed')
                }
            }
        }
        stage('Push to Dockerhub') {
            steps {
		        script {
                    withCredentials([usernamePassword(credentialsId: 'Docker_Cred_ID', passwordVariable: 'Docker_Password', usernameVariable: 'Docker_Name')]) {
                    sh 'docker login -u ${Docker_Name} -p ${Docker_Password}'
                    sh 'docker push ${Docker_Name}/netflix_clone:${BUILD_NUMBER}'
                    }
		        }
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "Netflix-DevSecOps"
                GIT_USER_NAME = "alphasknr"
            }
            steps {
                withCredentials([string(credentialsId: 'Github_Token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "krupasknr@gmail.com"
                        git config user.name "alphasknr"
                        sed -i "s/netflix-clone:.*/netflix=clone:${BUILD_NUMBER}/g" deployment.yaml
                        git add deployment.yaml
                        git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}