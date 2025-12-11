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
                echo 'Kod cekiliyor...'
                // Clean workspace before starting
                cleanWs()
                checkout scm
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo 'Docker image olusturuluyor...'
                echo 'Not: Bagimliliklar ve veri olusturma Docker icinde yapilacak.'
                sh 'docker build -t autogluon-iris .'
            }
        }
        
        stage('Run Training') {
            steps {
                echo 'Model egitimi baslatiliyor (Docker icinde)...'
                // Mount current directory to /app to save models/artifacts back to host
                sh 'docker run --rm -v ${PWD}:/app autogluon-iris python train.py'
            }
        }
        
        stage('MLSecOps Security Audit') {
            steps {
                echo '[GUVENLIK] Tum guvenlik testleri Docker icinde calistiriliyor...'
                sh 'docker run --rm -v ${PWD}:/app autogluon-iris python mlsecops_security.py'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'fairness_report.html, giskard_report.html, credo_model_card.md, sbom.json, vulnerability_report.json', allowEmptyArchive: true
                }
            }
        }
        
        stage('Detailed Checks (Optional)') {
             steps {
                echo 'Ekstra kontroller (Docker icinde)...'
                // Running these as separate steps just to show we can, using the same image
                sh 'docker run --rm -v ${PWD}:/app autogluon-iris python -c "from mlsecops_security import test_6_fairness_bias; test_6_fairness_bias()"'
                sh 'docker run --rm -v ${PWD}:/app autogluon-iris python -c "from mlsecops_security import test_7_giskard_validation; test_7_giskard_validation()"'
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

