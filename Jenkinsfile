pipeline {

    agent any

    environment {
        registry = "aksacrtcspoc.azurecr.io"
    }

    stages {

        stage ('checkout') {

            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jyothibasu/acr-aks-project.git']]])
            }
        }

        stage ('Code Analysis') {

            steps {
                sh '''mvn sonar:sonar \\
                -Dsonar.projectKey=Poc-AKS1 \\
                -Dsonar.host.url=http://52.140.116.20:9000 \\
                -Dsonar.login=e5ae4daa1e4c7cffe91ed213468a069ce581ff69'''
            }
        }

        stage ('Build') {

            steps {

                sh 'mvn clean package'
                
            }
        }

        stage ('Push Artifacts to Jfrog') {

            steps {
                sh '''cp webapp/target/webapp.war webapp/target/webapp_$BUILD_ID.war
                curl -uadmin:AP34mCp3r3nLNeLoHTaGnbrAuEJ -T webapp/target/webapp_$BUILD_ID.war "http://52.140.116.20:8081/artifactory/example-repo-local/"'''
                
            }
        }
        
        stage('ACR-Build & Push image'){
            steps {
                withCredentials([usernamePassword(credentialsId: 'acr_cred', passwordVariable: 'acrpswd', usernameVariable: 'aksacrtcspoc')]) {
                    sh "docker login aksacrtcspoc.azurecr.io -u $aksacrtcspoc -p $acrpswd"
                }
                sh '''docker build -t poc-1:v1.$BUILD_ID .
                docker tag poc-1:v1.$BUILD_ID $registry/poc-1:v1.$BUILD_ID
                docker push $registry/poc-1:v1.$BUILD_ID
                docker rmi poc-1:v1.$BUILD_ID
                docker rmi $registry/poc-1:v1.$BUILD_ID'''
            }
        }

        stage ('Deploy to AKS Cluster') {

            steps {
                script {
                    
                    kubernetesDeploy(
                    configs: 'acr-deployment.yaml',
                    kubeconfigId: 'k8s-cluster',
                    ) 
                }
                
            }
        }

    }
}
