pipeline {
    agent any

    tools {
        maven 'Maven3' 
        jdk 'Java17'   
    }

    parameters {
        booleanParam(name: 'RUN_STABILITY', defaultValue: true, description: 'Check this to run Code Stability Tests')
        booleanParam(name: 'RUN_QUALITY', defaultValue: true, description: 'Check this to run Code Quality Analysis')
        booleanParam(name: 'RUN_COVERAGE', defaultValue: true, description: 'Check this to run Code Coverage Analysis')
    }

    stages {
        stage('Code Checkout') {
            steps {
                // Since you are using "Pipeline script from SCM", Jenkins automatically checks out code.
                // But keeping this explicit step ensures compliance with your assignment stages.
                checkout scm
            }
        }
        
        stage('Parallel Code Execution') {
            parallel {
                stage('Code Stability') {
                    when { expression { return params.RUN_STABILITY } }
                    steps {
                        echo 'Running Unit Tests for Stability...'
                        sh 'mvn clean test'
                    }
                }
                
                stage('Code Quality Analysis') {
                    when { expression { return params.RUN_QUALITY } }
                    steps {
                        echo 'Running SonarQube Quality Scan...'
                        // Comment this out with // if you haven't configured SonarQube in Jenkins Global Settings yet
                        withSonarQubeEnv('SonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
                
                stage('Code Coverage Analysis') {
                    when { expression { return params.RUN_COVERAGE } }
                    steps {
                        echo 'Running Code Coverage (JaCoCo)...'
                        sh 'mvn org.jacoco:jacoco-maven-plugin:prepare-agent test org.jacoco:jacoco-maven-plugin:report'
                    }
                }
            }
        }
        
        stage('Generate Quality Report') {
            steps {
                echo 'Compiling the reports.....'
            }
        }
        
        stage('Approval Gate') {
            steps {
                input(message: 'Do you want to approve the publication of this artifact?', 
                      ok: 'Approve')
            }
        }
        
        stage('Publish Artifact') {
            steps {
                echo 'Publishing Artifact to repository.....'
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully'
            slackSend(channel: '#social', color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}] (${env.BUILD_URL})")
            emailext(to: 'vashishtha123456789@gmail.com',
                subject: "SUCCESS: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: "The build completed successfully. Check details here: ${env.BUILD_URL}")
        }
        failure {
            echo 'Pipeline Failed'
            slackSend(channel: '#social', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}] (${env.BUILD_URL})")
            emailext(to: 'vashishtha123456789@gmail.com',
                subject: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
                body: "The build failed. Check details here: ${env.BUILD_URL}")
        }
    }
}
