pipeline {
    agent any

    // Defining global tool environments makes sure commands like 'mvn' are recognized
    tools {
        maven 'maven' // Matches the name configured in Jenkins Global Tool Configuration
        jdk 'jdk21'   // Matches the JDK name configured in Jenkins
    }

    parameters {
        booleanParam(name: 'RUN_STABILITY', defaultValue: true, description: 'Check this to run Code Stability Tests')
        booleanParam(name: 'RUN_QUALITY', defaultValue: true, description: 'Check this to run Code Quality Analysis')
        booleanParam(name: 'RUN_COVERAGE', defaultValue: true, description: 'Check this to run Code Coverage Analysis')
    }

    stages {
        stage('Code Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Rishikvashu/Jenkins-Assignment-5.git'
            }
        }
        
        stage('Parallel Code Execution') {
            parallel {
                stage('Code Stability') {
                    when { expression { return params.RUN_STABILITY } }
                    steps {
                        echo 'Running Unit Tests for Stability...'
                        // mvn test runs JUnit tests. If any test fails, the build breaks here.
                        sh 'mvn clean test'
                    }
                }
                
                stage('Code Quality Analysis') {
                    when { expression { return params.RUN_QUALITY } }
                    steps {
                        echo 'Running SonarQube Quality Scan...'
                        // This block automatically handles authentication tokens configured in Jenkins
                        withSonarQubeEnv('SonarQube') {
                            sh 'mvn sonar:sonar'
                        }
                    }
                }
                
                stage('Code Coverage Analysis') {
                    when { expression { return params.RUN_COVERAGE } }
                    steps {
                        echo 'Running Code Coverage (JaCoCo)...'
                        // Generates the coverage data binary target/jacoco.exec and builds HTML/XML reports
                        sh 'mvn org.jacoco:jacoco-maven-plugin:prepare-agent test org.jacoco:jacoco-maven-plugin:report'
                    }
                }
            }
        }
        
        stage('Generate Quality Report') {
            steps {
                echo 'Publishing Quality and Coverage reports to Jenkins UI...'
                // Record checkstyle or code execution warnings if plugins are available
                // jacoco execPattern: 'target/*.exec', classPattern: '**/classes', sourcePattern: '**/src/main/java'
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
                echo 'Building and archiving final binary production JAR artifact...'
                // Compiles the production package without rerunning tests
                sh 'mvn package -DskipTests'
                
                // Archives the generated jar so you can download it directly from the Jenkins UI
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
        }
    }
}                body: "The build completed successfully. Check details here: ${env.BUILD_URL}")
                body: "The build failed. Check details here: ${env.BUILD_URL}")
            slackSend(channel: '#social', color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}] (${env.BUILD_URL})")
                subject: "FAILED: Job '${env.JOB_NAME}' [Build #${env.BUILD_NUMBER}]",
            emailext(to: 'vashishtha123456789@gmail.com',
            echo 'Pipeline Failed'

