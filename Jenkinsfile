pipeline {
    agent any
    tools{
        maven 'M2_HOME'
    }
    
    environment {
        SNAP_REPO = 'snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'yann42'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.0.35'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        registryCredential = 'ecr:us-east-1:awscreds'
        appRegistry = '510314780674.dkr.ecr.us-east-1.amazonaws.com/vproappimg'
        vprofileRegistry = 'https://510314780674.dkr.ecr.us-east-1.amazonaws.com'
    }

    
    stages{
        stage ('Build') {
            steps{
                sh 'mvn clean package'
            }    
        }

        stage ('Docker Image Build') {
            steps {
                script {
                    dockerImage = docker.build(appRegistry +":$BUILD_NUMBER", "./")
                }
            }
        }

        stage('Upload App Image') {
            steps {
                script{
                    docker.withRegistry (vprofileRegistry, registryCredential) {
                        dockerImage.push("$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Integrate Jenkins with EKS Cluster and deploy app') {
            steps {
                withAWS(credentials: 'awscreds', region: 'us-east-1')
                  script {
                    sh ('aws eks update-kubeconfig --name eks-cluster --region us-east-1')
                    sh "kubectl apply -f eks-deploy-k8s.yaml"
                }
                }   
            }
        }
    }

