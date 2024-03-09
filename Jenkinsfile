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
                sh "mvn clean verify sonar:sonar -Dsonar.projectKey=devsecops-sonarqube -Dsonar.projectName='devsecops-sonarqube' -Dsonar.host.url=http://35.197.23.84:9000 -Dsonar.token=sqp_b50609472c327e19573797cee59b19f5a389ffc1"
                
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
