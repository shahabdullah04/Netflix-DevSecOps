pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                // Pulling from YOUR GitHub repository now!
                git branch: 'main', url: 'https://github.com/shahabdullah04/Netflix-DevSecOps.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    // Matched the lowercase ID we created earlier
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
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
                   withDockerRegistry(credentialsId: 'docker'){   
                       // MUST REPLACE YOUR TMDB API KEY BELOW!
                       sh "docker build --build-arg TMDB_V3_API_KEY=<52cfd7ffd0eefc40babc55307f4e33f8> -t netflix ."
                       sh "docker tag netflix shahabdullah04/netflix:latest"
                       sh "docker push shahabdullah04/netflix:latest"
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image shahabdullah04/netflix:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d -p 8081:80 shahabdullah04/netflix:latest'
            }
        }
        
        // --- TEMPORARILY DISABLED FOR PHASE 3 ---
        // We will uncomment this in Phase 6 when we build the EKS Cluster
        /*
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes') {
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                                sh 'kubectl apply -f deployment.yml'
                                sh 'kubectl apply -f service.yml'
                        }   
                    }
                }
            }
        }
        */
    }
}