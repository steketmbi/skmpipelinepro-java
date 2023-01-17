pipeline{
    
    agent any
    
    stages {
        
        stage('Git Checkout'){
            
            steps{
                
                script{
                    
                    git branch: 'main', url: 'https://github.com/Mbianda2022/mrdevops_java_app.git'
                    
                }
            }
        }
        stage('Unit testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn test'
                    
                }
            }
        }
        stage('Itegration testing'){
            
            steps{
                
                script{
                    
                    sh 'mvn verify -DskipUnitTests'
                    
                }
            }
        }
        stage('Maven Build'){
            
            steps{
                
                script{
                    
                    sh 'mvn clean install'
                    
                }
            }
        }
        stage('Static code analysis'){
            
            steps{
                
                script{
                    
                 withSonarQubeEnv(credentialsId: 'sonar-api') {
                             
                      sh 'mvn clean package sonar:sonar'

                   }
                    
                }
            }
        }
        stage('Quality Gate Status'){
            
            steps{
                
                script{
                    
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-api'
                }
            }
        }
        stage('nexus artifact uploader'){

            steps{

                script{
                    def readPomVersion = readMavenPom file: 'pom.xml'

                    def nexusRepo = readPomVersion.version.endsWith("SNAPSHOT") ? "cloudlord-snapshot" :"cloudlord-releases"
                   
               nexusArtifactUploader artifacts: 
               [
                  [
                     artifactId: 'springboot', 
                     classifier: '', 
                     file: 'target/Uber.jar', 
                     type: 'jar'
                    ]
                ], 
                credentialsId: 'nexus-newcreds', 
                groupId: 'com.example', 
                nexusUrl: '44.201.155.173:8081', 
                nexusVersion: 'nexus3', 
                protocol: 'http', 
                repository: nexusRepo, 
                version: "${readPomVersion.version}"
                }
            }
        }
        stage('docker image building'){
            
            steps{
                
                script{
                    
                    sh 'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kubembianda/$JOB_NAME:v1.$BUILD_ID'
                    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID kubembianda/$JOB_NAME:latest'
                    
                }
            }
        }
        stage('Push image to DockerHub'){
            
            steps{
                
                script{
                    
                   withCredentials([string(credentialsId: 'cloudLord_dockerHub_pwd', variable: 'cloudlord_dockerHub_pwd')]) {
     
                        sh 'docker login -u kubembianda -p ${cloudlord_dockerHub_pwd}'
                        sh 'docker image push kubembianda/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker image push kubembianda/$JOB_NAME:latest'
                   
                         }
                   
                    } 
                }
            }
        }
    
    }
    
 

   
