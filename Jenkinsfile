pipeline{
    agent any
    
    tools {
        nodejs 'nodejs21'
    }
    
    environment{
        SONAR_HOME = tool 'sonar-scanner'
    }
    
    stages{
        stage("Git CheckOut") {
            steps{
                git branch: 'main', credentialsId: 'git-credential', url: 'https://github.com/Nikhil007nsg/3-Tier-Full-Stack.git'
            }
        }
        
        stage("Dependencies Installation"){
            steps{
                sh 'npm install'
            }
        }
        
        stage("Unit Test") {
            steps{
                sh 'npm test'
            }
        }
        
        stage("Trivy FS Scan"){
            steps{
                sh 'trivy fs --format table -o report.html .'
            }
        }
        
        stage("SonarQube"){
            steps{
                withSonarQubeEnv("sonar-scanner"){
                    sh '$SONAR_HOME/bin/sonar-scanner -Dsonar.projectKey=campground -Dsonar.projectName=campground'
                }
            }
        }
        
        stage("Docker Build & Tag"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                        sh 'docker build -t nikhilsg/campground:$BUILD_NUMBER .'
                    }
                }
            }
        }
        
        stage("Trivy Image Scan"){
            steps{
                sh 'trivy image --format table -o report.html nikhilsg/campground:$BUILD_NUMBER'
            }
        }
        
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                        sh 'docker push nikhilsg/campground:$BUILD_NUMBER'
                    }
                }
            }
        }
         stage('Upload Deployment File') {
            environment {
                GIT_REPO_NAME = "3-Tier-Full-Stack"
                GIT_USER_NAME = "nikhil007nsg"
            }
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        git config user.email "nikhil007nsg@gmail.com"
                        git config user.name "nikhil007nsg"
                        BUILD_NUMBER=${BUILD_NUMBER}
                        cp Manifests/dss.yaml Production/
                        sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Production/dss.yaml
                        git add Production/
                        git commit -m "Update Deployment Manifest for Production"
                        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
     
    }
}
