pipeline {
    agent any

    environment {
        PROJECT_NAME = "test-dt-alert"
        PROJECT_VERSION = "1.0.4"
        BOM_PATH = "bom.json"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/koojennie/test-dt-alert.git',
                    credentialsId: 'github-creds'
            }
        }

        stage('Install & Generate SBOM') {
            steps {
                sh '''
                  echo "=== Installing dependencies & generating SBOM ==="
                  npm install
                  npx cyclonedx-npm \
                    --flatten-components \
                    --output-format json \
                    --output-file bom.json
                '''
            }
        }

        stage('Publish BOM to Dependency-Track') {
            steps {
                withCredentials([
                    string(credentialsId: 'dt-api-key', variable: 'DTRACK_API_KEY')
                ]) {
                    dependencyTrackPublisher(
                        artifact: "${BOM_PATH}",
                        projectName: "${PROJECT_NAME}",
                        projectVersion: "${PROJECT_VERSION}",
                        synchronous: true,
                        dependencyTrackApiKey: DTRACK_API_KEY
                    )
                }
            }
        }
    }

    post {
        success {
            echo 'SBOM uploaded successfully to Dependency-Track'
        }
        failure {
            echo 'Oh no! You got an Error. Pipeline failed'
        }
    }
}
