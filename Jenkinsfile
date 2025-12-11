pipeline {
    agent any
    
    environment {
        DOCKER_BUILDKIT = '1'
        // Workspace directory for volume mounting
        WORKSPACE_DIR = "${WORKSPACE}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                cleanWs()
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Docker image olusturuluyor...'
                sh 'docker build -t autogluon-iris .'
            }
        }
        
        stage('Run Pipeline (Train & Secure)') {
            steps {
                script {
                    try {
                        echo 'Pipeline container icinde baslatiliyor...'
                        // Remove old container if exists
                        sh 'docker rm -f runner || true'
                        
                        // Run training and security checks in the SAME container sequentially
                        // keeping state (models) between steps.
                        // No volume mounts to avoid DinD path issues.
                        sh '''
                            docker run --name runner autogluon-iris /bin/sh -c " \
                                echo '--- 1. Training Model ---' && \
                                python train.py && \
                                echo '--- 2. Security Audit ---' && \
                                python mlsecops_security.py \
                            "
                        '''
                    } finally {
                        echo 'Artifacts kopyalaniyor...'
                        // Copy reports out of the container even if it failed midway
                        sh 'docker cp runner:/app/fairness_report.html . || true'
                        sh 'docker cp runner:/app/giskard_report.html . || true'
                        sh 'docker cp runner:/app/credo_model_card.md . || true'
                        sh 'docker cp runner:/app/sbom.json . || true'
                        sh 'docker cp runner:/app/vulnerability_report.json . || true'
                        sh 'docker cp runner:/app/mlflow.db . || true'
                        sh 'docker cp runner:/app/mlruns . || true'
                        
                        echo 'Temizlik yapiliyor...'
                        sh 'docker rm -f runner'
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '*.html, *.md, *.json, mlflow.db, mlruns/**/*', allowEmptyArchive: true
                }
            }
        }
    }
    
    post {
        success {
            echo '[OK] Pipeline basariyla tamamlandi!'
            echo 'Artifacts have been archived.'
        }
        failure {
            echo '[FAIL] Pipeline basarisiz oldu!'
        }
    }
}

