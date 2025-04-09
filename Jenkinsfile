pipeline {
    agent any

    environment {
        MAVEN_OPTS = '-Dmaven.test.failure.ignore=true'
    }

    stages {
        stage('Detect changed service') {
            steps {
                script {
                    // Kiểm tra thay đổi trong service nào
                    def changedFiles = sh(script: "git diff --name-only origin/main...HEAD", returnStdout: true).trim().split("\n")
                    def services = ['customers-service', 'vets-service', 'visit-service', 'admin-server', 'api-gateway', 'eureka-server']
                    env.SERVICES_TO_BUILD = services.findAll { service ->
                        changedFiles.any { it.startsWith(service + "/") }
                    }.join(',')
                    echo "Services changed: ${env.SERVICES_TO_BUILD}"
                }
            }
        }

        stage('Build & Test') {
            when {
                expression { env.SERVICES_TO_BUILD }
            }
            steps {
                script {
                    env.SERVICES_TO_BUILD.split(',').each { service ->
                        echo "Building and testing ${service}"
                        dir(service) {
                            sh 'mvn clean verify'
                        }
                    }
                }
            }
        }

        stage('Publish Test Results') {
            when {
                expression { env.SERVICES_TO_BUILD }
            }
            steps {
                junit '**/target/surefire-reports/*.xml'
                cobertura coberturaReportFile: '**/target/site/cobertura/coverage.xml'
            }
        }

        stage('Check coverage > 70%') {
            when {
                expression { env.SERVICES_TO_BUILD }
            }
            steps {
                script {
                    def coverage = coberturaAutoUpdateThreshold()
                    if (coverage < 70.0) {
                        error("❌ Coverage is too low: ${coverage}% < 70%")
                    } else {
                        echo "✅ Coverage passed: ${coverage}%"
                    }
                }
            }
        }
    }
}
