pipeline {
    agent any
    environment {
        CHANGED_SERVICES = 'vets-service'
    }

    stages {
        stage('Detect changed service') {
            steps {
                script {
                    // Detect các file thay đổi so với main branch
                    def changes = sh(script: "git diff --name-only origin/main...HEAD", returnStdout: true).trim().split("\n")
                    echo "Changed files:\n${changes}"

                    // Xác định service thay đổi dựa trên tên thư mục cấp 1
                    def services = []
                    for (change in changes) {
                        def topFolder = change.tokenize('/')[0]
                        if (["customers-service", "vets-service", "visits-service"].contains(topFolder)) {
                            services << topFolder
                        }
                    }

                    services = services.unique()
                    CHANGED_SERVICES = services.join(',')

                    echo "Services changed: ${CHANGED_SERVICES}"
                }
            }
        }

        stage('Build & Test') {
            when {
                expression { return CHANGED_SERVICES?.trim() }
            }
            steps {
                script {
                    for (svc in CHANGED_SERVICES.split(',')) {
                        echo "Building and testing service: ${svc}"
                        dir("${svc}") {
                            sh "../../mvnw clean test"
                        }
                    }
                }
            }
        }

        stage('Publish Test Results') {
            when {
                expression { return CHANGED_SERVICES?.trim() }
            }
            steps {
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Check coverage > 70%') {
            when {
                expression { return CHANGED_SERVICES?.trim() }
            }
            steps {
                echo "Checking code coverage (placeholder, cần thêm plugin hoặc báo cáo coverage)..."
            }
        }
    }

    post {
        always {
            echo "Finished pipeline execution"
        }
    }
}
