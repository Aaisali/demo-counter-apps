pipeline{
    
    agent any 
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/Aaisali/demo-counter-apps.git'
                }
            }
        }
        stage('UNIT testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                }
            }
        }
        stage('Integration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                }
            }
        }

        stage('Maven build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                }
            }
        }

        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                    withSonarQubeEnv(credentialsId: 'sonar-qwe') {
                        
                        sh 'mvn clean package sonar:sonar'
                    }
                   }
                    
                }
            }
    
        stage('Quality Gate Status'){
                
                steps{
                    
                    script{
                        
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonar-qwe'
                    }
                }
            }
        stage('Upload war file to nexus'){

            steps{
                script{
                    def readpomVersion = readMavenPom file: 'pom.xml'
                    def nexusRepo = readpomVersion.version.endsWith("SNAPSHOT") ? "demoapp-snapshot" : "demoapp-release"

                    nexusArtifactUploader artifacts: 
                    [
                        [
                          artifactId: 'springboot', 
                          classifier: '', file: 'target/Uber.jar', 
                          type: 'jar'
                        ]
                    ], 
                    credentialsId: 'nexus-auth', 
                    groupId: 'com.example', 
                    nexusUrl: '54.89.250.225:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepo, 
                    version: "${readpomVersion.version}"
                }
            }
        }

        stage('Docker Image Build') {
            steps{
                script{
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID aaisali/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID aaisali/$JOB_NAME:latest'
                }
            }
        }

        stage('push image to docker hub'){
            steps{
                script{
                    withCredentials([string(credentialsId: 'git_creds', variable: 'docker_hub_cred')]) {
                        sh 'docker login -u aaisali -p ${docker_hub_cred}'
                        sh 'docker image push aaisali/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image push aaisali/$JOB_NAME:latest'
                }
                }

            }
        }   
    }            
        
}

