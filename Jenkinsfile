pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
        
    }
    
    environment {
        
        SCANNER_HOME=tool 'sonar-scanner' 
    }

    stages {
        stage('Clear WorkSpace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kundan344/Netflix-Clone-Deployment-on-EKS-Using-Terraform-Jenkins-AWS-DevSecOps.git'
            }
        }
        
        stage('Code Analysis by SonaQube') {
            steps {
                withSonarQubeEnv('sonar') {
                  sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=netflix \
                  -Dsonar.projectKey=netfilx'''
              }
            }
        }
        
        stage('Dependencies Installation') {
            steps {
                sh "npm install"
            }
        }
        
        stage('Dependencies Check by OWASP') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'dc'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Triy FS Can') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.xml ."
            }
        }
        
        stage('Docker Image Build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t kundankumar344/netflix:latest ."
                  }
                }
            }
        }
        
        stage('Docker Images Scan By Trivy') {
            steps {
                sh "trivy image --format table -o trivy-docker-image-report.xml kundankumar344/netflix:latest"
            }
        }
        
        stage('Docker Image Push in DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push kundankumar344/netflix:latest"
                  }
                }
            }
        }
        
        stage('Deployment On EKS') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8s-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.29.62:6443') {
                   sh "kubectl apply -f deploymentservice.yml -n webapps"
				   sh "kubectl get pods -n webapps"
				   sh "kubectl get svc -n webapps"
               }
            }
        }
    
    }
}
