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
        
        stage('Setup Container') {
            steps {
                script {
                    echo 'Pipeline container baslatiliyor...'
                    sh 'docker rm -f runner || true'
                    // Start container in background (detached)
                    sh 'docker run -d --name runner --entrypoint tail autogluon-iris -f /dev/null'
                }
            }
        }

        stage('Training Model') {
            steps {
                echo '--- 1. Training Model ---'
                sh 'docker exec runner python train.py'
            }
        }

        stage('Security Audit') {
            steps {
                echo '--- 2. Security Audit ---'
                sh 'docker exec runner python mlsecops_security.py'
            }
        }

        stage('Collect Artifacts') {
            steps {
                echo 'Artifacts kopyalaniyor...'
                sh 'docker cp runner:/app/fairness_report.html . || true'
                sh 'docker cp runner:/app/giskard_report.html . || true'
                sh 'docker cp runner:/app/credo_model_card.md . || true'
                sh 'docker cp runner:/app/sbom.json . || true'
                sh 'docker cp runner:/app/vulnerability_report.json . || true'
                sh 'docker cp runner:/app/mlflow.db . || true'
                sh 'docker cp runner:/app/mlruns . || true'
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Temizlik yapiliyor...'
                sh 'docker rm -f runner || true'
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

