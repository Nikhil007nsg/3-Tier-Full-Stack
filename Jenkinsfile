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
        
        stage("Docker Deploy To Dev Env"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker'){
                        sh 'docker run -d --name camp3 -p 3000:3000 nikhilsg/campground:$BUILD_NUMBER'
                    }
                }
            }
        }
    }
}
