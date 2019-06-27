pipeline {

    agent none

    environment {
        imageName = "app/spring-boot-example"
        dockerRegistryCredential = "registry-ci-yourdomain-com-credentials"
        dockerRegistryUrl = "https://registry.ci.yourdomain.com"
        kubernetesClusterCredentials = "kubernetes-ci-yourdomain-com-credentials"
        kubernetesClusterUrl = "https://yourdomain-d37f-4850-9a5a-96e0b882b13a.k8s.ondigitalocean.com"
        kubernetesClusterNamespace = "production"
    }

    stages {
        stage('Jar') {
            agent {
                docker {
                    image 'openjdk:8-jdk-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh './mvnw clean package'
            }
        }
        stage('Docker') {
            agent {
                docker {
                    image 'docker'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                  docker.withRegistry(dockerRegistryUrl, dockerRegistryCredential ) {
                    docker.build(imageName + ":$GIT_COMMIT").push()
                  }
                }
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'dtzar/helm-kubectl'
                }
            }
            steps {
                script {
                    withKubeConfig([credentialsId: kubernetesClusterCredentials, namespace: kubernetesClusterNamespace, serverUrl: kubernetesClusterUrl]) {
                      sh 'sed s%SPRING_BOOT_EXAMPLE_VERSION%$GIT_COMMIT% deployment.yaml | kubectl apply -f - --record'
                    }
                }
            }
        }
    }
}
