pipeline {
    agent any
    environment {
        CHANGED_SERVICES = ''
    }

    stages {
        stage('Prepare') {
            steps {
                // Fetch full history để có thể so sánh branch
                sh 'git fetch --all'
            }
        }

        stage('Detect changed service') {
            steps {
                script {
                    // Detect các file thay đổi so với main branch
                    def changes = sh(
                        script: "git diff --name-only origin/main HEAD -- . ':!Jenkinsfile'", 
                        returnStdout: true
                    ).trim()
                    
                    echo "Changed files:\n${changes}"
                    
                    // Xác định service thay đổi
                    def services = []
                    if (changes) {
                        changes.split('\n').each { change ->
                            def topFolder = change.split('/')[0]
                            if (["customers-service", "vets-service", "visits-service"].contains(topFolder)) {
                                services << topFolder
                            }
                        }
                        services = services.unique()
                        env.CHANGED_SERVICES = services.join(',')
                    }
                    
                    echo "Services changed: ${env.CHANGED_SERVICES}"
                }
            }
        }

        stage('Build & Test') {
            when {
                expression { return env.CHANGED_SERVICES?.trim() }
            }
            steps {
                script {
                    env.CHANGED_SERVICES.split(',').each { svc ->
                        echo "Building and testing service: ${svc}"
                        dir(svc) {
                            // Sử dụng Maven wrapper trong thư mục service
                            sh "./mvnw clean test" 
                        }
                    }
                }
            }
        }

        stage('Publish Test Results') {
            when {
                expression { return env.CHANGED_SERVICES?.trim() }
            }
            steps {
                junit '**/target/surefire-reports/*.xml'
            }
        }

        stage('Check coverage > 70%') {
            when {
                expression { return env.CHANGED_SERVICES?.trim() }
            }
            steps {
                cobertura coberturaReportFile: '**/target/site/jacoco/jacoco.xml'
            }
        }
    }

    post {
        always {
            echo "Finished pipeline execution"
        }
    }
}