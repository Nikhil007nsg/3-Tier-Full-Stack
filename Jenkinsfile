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
                 //       sh 'docker build -t nikhilsg/campground:$BUILD_NUMBER .'
                         sh 'docker build -t nikhilsg/campground:latest .'
                    }
                }
            }
        }
        
        stage("Trivy Image Scan"){
            steps{
              //  sh 'trivy image --format table -o report.html nikhilsg/campground:$BUILD_NUMBER'
                
                sh 'trivy image --format table -o report.html nikhilsg/campground:latest'
            }
        }
        
        stage("Docker Push"){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker-crd', toolName: 'docker') {
                       // sh 'docker push nikhilsg/campground:$BUILD_NUMBER'
                         sh 'docker push nikhilsg/campground:latest'
                    }
                }
            }
        }
 //        stage('Upload Deployment File') {
   //         environment {
    //            GIT_REPO_NAME = "3-Tier-Full-Stack"
     //           GIT_USER_NAME = "nikhil007nsg"
      //      }
       //     steps {
        //        withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
         //           sh '''
          //              git config user.email "nikhil007nsg@gmail.com"
           //             git config user.name "nikhil007nsg"
            //            BUILD_NUMBER=${BUILD_NUMBER}
             //           mkdir -p Production
              //          cp Manifests/dss.yml Production/
               //         sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" Production/dss.yml
                //        git add Production/
                 //       git commit -m "Update Deployment Manifest for Production"
                //        git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                //    '''
          //      }
     //       }
   //     }
        stage('Deploy to EKS') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://852C305A54A18CF2AEC884C18727E0B1.gr7.ap-south-1.eks.amazonaws.com']]) {
                  sh "kubectl apply --validate=true -f Manifests/dss.yml"
                  sleep 60
}
}
}
         stage('verify the Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'eks-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://852C305A54A18CF2AEC884C18727E0B1.gr7.ap-south-1.eks.amazonaws.com']]) {
                  sh "kubectl get pods -n webapps"
                  sh "kubectl get svc -n webapps"
}
}
}
        
        
    }
}
