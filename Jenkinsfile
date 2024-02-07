// Imported shared library
@Library('my-shared-library') _

pipeline {
    agent any

    parameters {
        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "Name of the Docker image build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "Tag of the Docker image build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "DockerHub username", defaultValue: 'karthikreddy432')
    }

    stages {
        // Stage to checkout code from Git
        stage('Git Checkout') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                gitCheckout(
                    branch: "main",
                    url: "https://github.com/karthikreddy432/Java_app_3.0.git"
                )
            }
        }

        // Stage for running unit tests with Maven
        stage('Unit Test maven') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    mvnTest()
                }
            }
        }

        // Stage for running integration tests with Maven
        stage('Integration Test maven') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    mvnIntegrationTest()
                }
            }
        }

        // Stage for static code analysis with Sonarqube
        stage('Static code analysis: Sonarqube') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    statiCodeAnalysis(SonarQubecredentialsId)
                }
            }
        }

        // Stage for checking quality gate status with Sonarqube
        stage('Quality Gate Status Check : Sonarqube') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    def SonarQubecredentialsId = 'sonarqube-api'
                    QualityGateStatus(SonarQubecredentialsId)
                }
            }
        }

        // Stage for building the Maven project
        stage('Maven Build : maven') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    mvnBuild()
                }
            }
        }

        // Stage for pushing artifacts to JFrog Artifactory
        stage('Push to JFrog Artifactory') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    // Connect to JFrog Artifactory server
                    def server = Artifactory.server 'Pushartifact'
                    // Define the upload specification
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "example-repo-local/"
                            }
                        ]
                    }"""
                    // Upload artifacts to JFrog Artifactory
                    server.upload(uploadSpec)
                }
            }
        }

        // Stage for building Docker image
        stage('Docker Image Build') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        // Stage for scanning Docker image with trivy
        stage('Docker Image Scan: trivy') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        // Stage for pushing Docker image to DockerHub
        stage('Docker Image Push : DockerHub') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }

        // Stage for cleaning up Docker image from DockerHub
        stage('Docker Image Cleanup : DockerHub') {
            when { 
                expression { params.action == 'create' } 
            }
            steps {
                script {
                    dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
                }
            }
        }
    }
}
