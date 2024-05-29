pipeline{
 agent any
//  tools{
//      jdk 'jdk17'
//      nodejs 'node16'
//  }
 environment {
        SCANNER_HOME = tool "sonar"
    }
    
    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage("Checkout Code from git"){
            steps{
                git branch: 'main', url: 'https://github.com/Satish9422/DevSecOps-on-EKS.git'
            }
            
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar') {
                    sh ' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix -Dsonar.projectKey=Netflix '
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=b9c7f1f23c53a4a1f6e551b3ff9869b7 -t netflix ."
                       sh "docker tag netflix satish680/netflix:latest "
                       sh "docker push satish680/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image satish680/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 satish680/netflix:latest'
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: 'eks-clusters', contextName: 'my-eks-context', credentialsId: 'k8s', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://BC16A0120529C26B15E94F4E0C25A47C.gr7.ap-south-1.eks.amazonaws.com') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
    }
    
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
                "Build Number: ${env.BUILD_NUMBER}<br/>" +
                "URL: ${env.BUILD_URL}<br/>",
            to: 'satishsuryawanshi303@gmail.com',
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
