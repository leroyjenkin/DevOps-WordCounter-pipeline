pipeline {
    agent any

    stages {
        stage('Configure') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/leroyjenkin/Word-counter']])
            }
        }
        stage('Building image') {
            steps {
                sh 'docker build -t word-counter .'
            }
        }
        
        stage('Pushing to ECR') {
            steps {
                withAWS(credentials: '90ef7f7a-a4e3-48ce-9e0a-2ecfb25ca894', region: 'eu-central-1') {
                    sh 'aws ecr get-login-password --region eu-central-1 | docker login --username AWS --password-stdin 703671940174.dkr.ecr.eu-central-1.amazonaws.com/word-counter'
                    sh 'docker tag word-counter:latest 703671940174.dkr.ecr.eu-central-1.amazonaws.com/word-counter'
                    sh 'docker push 703671940174.dkr.ecr.eu-central-1.amazonaws.com/word-counter'
                }
            }
        }
          
        stage('K8S Deploy') {
            steps {
                script {
                    withAWS(credentials: '90ef7f7a-a4e3-48ce-9e0a-2ecfb25ca894', region: 'eu-central-1') {
                        sh 'aws eks update-kubeconfig --name education-eks-ZBfKt5kw --region eu-central-1'
                        sh 'kubectl apply -f EKS-Deployment.yaml'
                    }
                }
            }
        }

        stage('K8S Deploy') {

            steps {

                script {

                    withAWS(credentials: '90ef7f7a-a4e3-48ce-9e0a-2ecfb25ca894', region: 'eu-central-1') {

                        sh 'aws eks update-kubeconfig --name education-eks-ZBfKt5kw --region eu-central-1'

                        sh 'kubectl apply -f EKS-Deployment.yaml'

                    }

                }

            }

        }


        stage('Get Service URL') {

            steps {

                script {

                    def serviceUrl = ""

                    // Wait for the LoadBalancer IP to be assigned

                    timeout(time: 5, unit: 'MINUTES') {

                        while(serviceUrl == "") {

                            serviceUrl = sh(script: "kubectl get svc word-counter-service -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()

                            if(serviceUrl == "") {

                                echo "Waiting for the LoadBalancer IP..."

                                sleep 10

                            }

                        }

                    }

                    echo "Service URL: http://${serviceUrl}"

                }

            }

        }

    }

} 




