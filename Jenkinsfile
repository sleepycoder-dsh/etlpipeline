pipeline {
    agent any

    environment {
        // Credentials created in Jenkins (Manage Jenkins → Credentials → Global)
        DB_HOST  = credentials('DATABRICKS_HOST')
        DB_TOKEN = credentials('DATABRICKS_TOKEN')
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo '=== Checking out code from GitHub... ==='
                git branch: 'main', url: 'https://github.com/sleepycoder-dsh/etlpipeline.git'
            }
        }

        stage('Configure Databricks CLI') {
            steps {
                echo '=== Configuring Databricks CLI... ==='
                sh '''
                set -x
                mkdir -p /var/lib/jenkins
                echo "[DEFAULT]" > /var/lib/jenkins/.databrickscfg
                echo "host = ${DB_HOST}" >> /var/lib/jenkins/.databrickscfg
                echo "token = ${DB_TOKEN}" >> /var/lib/jenkins/.databrickscfg
                chmod 600 /var/lib/jenkins/.databrickscfg
                set +x
                echo "Databricks CLI configuration complete."
                '''
            }
        }

        stage('Deploy Notebook to Databricks') {
            steps {
                echo '=== Uploading Bookhive_ETL notebook to Databricks Workspace... ==='
                sh '''
                databricks workspace import \
                    ./Bookhive_ETL.ipynb \
                    /Shared/Bookhive_ETL \
                    --format JUPYTER \
                    --overwrite
                '''
            }
        }

        stage('Run ETL Job in Databricks') {
            steps {
                echo '=== Triggering ETL Job on Databricks ==='
                sh '''
                databricks jobs run-now --job-id <YOUR_JOB_ID>
                '''
            }
        }
    }

    post {
        success {
            echo ' ETL Pipeline executed successfully!'
        }
        failure {
            echo ' ETL Pipeline failed. Please check logs in Databricks.'
        }
    }
}
