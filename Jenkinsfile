pipeline {
    agent any
    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', credentialsId: '151f7e96-d625-449b-a949-f76315919e85', url: 'https://github.com/abdulfayis681/gcp-infrastructure-terraform.git'
            }
        }

        stage('Install Checkov') {
            steps {
                script {
                    sh "sudo pip install checkov"
                    def checkovPath = sh(script: 'pip show checkov | grep "Location" | cut -d " " -f 2', returnStdout: true).trim()
                    env.PATH = "${checkovPath}:${env.PATH}"
                }
            }
        }
        stage('Initialize') {
            steps {
                sh 'terraform init'
            }
        }
        stage('checkov scan ') {
            steps {
                catchError(buildResult: 'SUCCESS') {
                    script {
                        try {
                            sh 'sudo mkdir -p reports'
                            sh 'sudo checkov -d . --output junitxml > reports/checkov-report.xml'
                            junit skipPublishingChecks: true, testResults: 'reports/checkov-report.xml'
                        } catch (err) {
                            junit skipPublishingChecks: true, testResults: 'reports/checkov-report.xml'
                            throw err
                        }
                        
                    }
                }
            }
        }
        
        stage('Trivy scan ') {
            steps {
                sh 'sudo curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'
                sh 'sudo mkdir -p reports'
                sh 'sudo terraform plan --out tfplan.binary'
                sh 'sudo terraform show -json tfplan.binary > tfplan.json'
                sh 'sudo trivy config ./tfplan.json --format template --template @./html.tpl -o reports/trivy-report.html '
                sh 'trivy fs . > trivyfs.txt'
        
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'trivy-report.html',
                    reportName: 'Trivy Scan',
                    reportTitles: 'Trivy Scan'
                ]
            }
        }

        stage('Plan') {
            steps {
                sh 'terraform plan'
            }
        }

        stage('Apply') {
            steps {
                script {
                    sh 'terraform apply -auto-approve'
                }
            }
        }

    }


}
