pipeline {
    agent any

    environment {
    DB_TOKEN = credentials('DATABRICKS')
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
            withCredentials([
        string(credentialsId: 'databricks-host', variable: 'DB_HOST'),
        string(credentialsId: 'DATABRICKS', variable: 'DB_TOKEN')
    ]) {
        sh '''
            echo "[DEFAULT]" > /var/lib/jenkins/.databrickscfg
            echo "host = ${DB_HOST}" >> /var/lib/jenkins/.databrickscfg
            echo "token = ${DB_TOKEN}" >> /var/lib/jenkins/.databrickscfg
        '''
    }

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
                    --language PYTHON \
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
