pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
        disableConcurrentBuilds()
        durabilityHint('PERFORMANCE_OPTIMIZED')
        preserveStashes(buildCount: 5)
    }

    triggers {
        githubPush()
    }

    environment {
        SHORT_SHA = "${env.GIT_COMMIT.take(7)}"
    }

    stages {
        stage('Detect Changes') {
            steps {
                script {
                    // Get changed files vs previous commit
                    def changedFiles = sh(
                        script: "git diff --name-only HEAD~1 HEAD",
                        returnStdout: true
                    ).trim().split("\n")

                    // Set flags
                    env.BUILD_FRONTEND = changedFiles.any { it.startsWith('frontend/') }.toString()
                    env.BUILD_BACKEND = changedFiles.any { it.startsWith('backend/') }.toString()

                    echo "Services to build:"
                    if (env.BUILD_FRONTEND == "true") {
                        echo "- Frontend"
                    }
                    if (env.BUILD_BACKEND == "true") {
                        echo "- Backend"
                    }
                }
            }
        }

        stage('Build Services') {
            parallel {
                stage('Frontend') {
                    when {
                        expression { env.BUILD_FRONTEND == "true" }
                    }
                    steps {
                        build job: 'frontend', parameters: [
                            string(name: 'GIT_BRANCH', value: env.GIT_BRANCH),
                            string(name: 'SHORT_SHA', value: env.SHORT_SHA),
                        ]
                    }
                }

                stage('Backend') {
                    when {
                        expression { env.BUILD_BACKEND == "true" }
                    }
                    steps {
                        build job: 'backend', parameters: [
                            string(name: 'GIT_BRANCH', value: env.GIT_BRANCH),
                            string(name: 'SHORT_SHA', value: env.SHORT_SHA),
                        ]
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
        success { echo '✅ Pipeline succeeded!' }
        failure { echo '❌ Pipeline failed!' }
    }
}
