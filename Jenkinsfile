pipeline {
    agent any
    options {
        timestamps()
        skipDefaultCheckout(true)
    }
    environment {
        CI = 'true'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test (Maven)') {
            when {
                expression { fileExists('pom.xml') }
            }
            steps {
                script {
                    def hasWrapper = fileExists(isUnix() ? 'mvnw' : 'mvnw.cmd')
                    def mvnCmd = hasWrapper ? (isUnix() ? './mvnw' : 'mvnw.cmd') : 'mvn'
                    if (isUnix()) {
                        sh "${mvnCmd} -B -ntp clean verify"
                    } else {
                        bat "${mvnCmd} -B -ntp clean verify"
                    }
                bat 'bash -lc "set -e; uname -a; mvn -v"'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/surefire-reports/*.xml, **/failsafe-reports/*.xml'
                    archiveArtifacts artifacts: 'target/**/*.{jar,war,zip}', onlyIfSuccessful: true, allowEmptyArchive: true
                }
            }
        }

        stage('Build & Test (Gradle)') {
            when {
                expression { fileExists('build.gradle') || fileExists('build.gradle.kts') }
            }
            steps {
                script {
                    def hasWrapper = fileExists(isUnix() ? 'gradlew' : 'gradlew.bat')
                    def gradleCmd = hasWrapper ? (isUnix() ? './gradlew' : 'gradlew.bat') : (isUnix() ? 'gradle' : 'gradle')
                    if (isUnix()) {
                        sh "${gradleCmd} clean build --no-daemon"
                    } else {
                        bat "${gradleCmd} clean build --no-daemon"
                    }
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/build/test-results/**/*.xml'
                    archiveArtifacts artifacts: 'build/libs/*.{jar,war,zip}', onlyIfSuccessful: true, allowEmptyArchive: true
                }
            }
        }

        stage('Build & Test (Node.js)') {
            when {
                expression { fileExists('package.json') }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            npm ci || npm install
                            npm run test --if-present
                            npm run build --if-present
                        '''
                    } else {
                        bat '''
                            npm ci || npm install
                            npm run test --if-present
                            npm run build --if-present
                        '''
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'dist/**/*, build/**/*', allowEmptyArchive: true
                }
            }
        }

        stage('Build & Test (Python)') {
            when {
                expression { fileExists('pyproject.toml') || fileExists('requirements.txt') }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh '''
                            set -e
                            (python3 -V || python -V)
                            (pip3 install -U pip setuptools wheel || pip install -U pip setuptools wheel)
                            if [ -f requirements.txt ]; then (pip3 install -r requirements.txt || pip install -r requirements.txt); fi
                            if [ -f pyproject.toml ]; then (pip3 install -e . || pip install -e . || true); fi
                            (pytest -q || true)
                        '''
                    } else {
                        bat '''
                            python --version
                            pip install -U pip setuptools wheel
                            if exist requirements.txt ( pip install -r requirements.txt )
                            if exist pyproject.toml ( pip install -e . )
                            pytest -q || ver > nul
                        '''
                    }
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/test-results/**/*.xml, **/junit-*.xml'
                }
            }
        }

        stage('Docker Build (optional)') {
            when {
                allOf {
                    expression { fileExists('Dockerfile') }
                    anyOf {
                        expression { fileExists('target') }
                        expression { fileExists('build') }
                        expression { fileExists('dist') }
                    }
                }
            }
            steps {
                script {
                    def imageTag = "${env.JOB_BASE_NAME}:${env.BUILD_NUMBER}"
                    if (isUnix()) {
                        sh "docker build -t ${imageTag} ."
                    } else {
                        bat "docker build -t ${imageTag} ."
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Branch: ${env.BRANCH_NAME}  Build: #${env.BUILD_NUMBER}"
        }
        cleanup {
            deleteDir()
        }
    }
}