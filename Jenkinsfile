pipeline {
    agent any

    stages {
        stage('Build Artifact') {
            steps {
                sh "mvn clean package -DskipTests=true"
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('SonarQube - SAST Analysis') {
            steps {
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-sonarqube -Dsonar.host.url=http://35.197.23.84:9000 -Dsonar.login=sqp_02f168728a2b7ff1a6992a8c3561a59038206a2e"
                
            }
        }

        stage('Docker Build and Push') {
            steps {
                withDockerRegistry(credentialsId: "docker-hub", url: "https://index.docker.io/v1/") {
                    sh "printenv"
                    sh "docker build -t dsocouncil/node-service:v1 ."
                    sh "docker push dsocouncil/node-service:v1"
                }
            }
        }

        stage('Kubernetes Deployment - DEV') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "sed -i 's#replace#dsocouncil/node-service:v1${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                    sh "kubectl apply -f k8s_deployment_service.yaml"
                }
            }
        }
    }
}
