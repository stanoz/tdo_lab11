pipeline {
    agent any

    environment {
        TARGET_DIR = 'target'
        CLASS_DIR = "${TARGET_DIR}/classes"
        TEST_DIR = "${TARGET_DIR}/test-classes"
        REPORT_DIR = "${TARGET_DIR}/reports"
    }

    stages {
         stage('Checkout') {
                    steps {
                        git url: 'https://github.com/Mredosz/Lab11TDO.git', branch: 'main'
                    }
         }

        stage('Validation') {
            steps {
                script {
                    def requiredDirs = ['src/main/java', 'src/test/java', 'lib']
                    for (dir in requiredDirs) {
                        if (!fileExists(dir)) {
                            error "Wymagany katalog '${dir}' nie istnieje. Przerywam pipeline."
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh '''
                    mkdir -p ${CLASS_DIR} ${TEST_DIR} ${REPORT_DIR}

                    javac -cp "lib/*" -d ${CLASS_DIR} $(find src/main/java -name "*.java")
                    javac -cp "${CLASS_DIR}:lib/*" -d ${TEST_DIR} $(find src/test/java -name "*.java")
                '''
            }
            post {
                success {
                    stash includes: 'target/classes/**,target/test-classes/**,lib/**', name: 'compiled'
                }
            }
        }

        stage('Test') {
            steps {
                unstash 'compiled'
                script {
                    try {
                        sh '''
                            java -jar lib/junit-platform-console-standalone-*.jar \
                              --class-path ${CLASS_DIR}:${TEST_DIR}:lib/* \
                              --scan-class-path \
                              --reports-dir=${REPORT_DIR}
                        '''
                    } finally {
                        junit "${REPORT_DIR}/*.xml"
                    }
                }
            }
        }

        stage('Package') {
            when {
                branch 'main'
            }
            steps {
                sh "jar cf app-${BUILD_ID}.jar -C ${CLASS_DIR} ."
            }
        }

        stage('Archive') {
            when {
                branch 'main'
            }
            steps {
                archiveArtifacts artifacts: "app-${BUILD_ID}.jar, ${REPORT_DIR}/*.xml", onlyIfSuccessful: true
            }
        }
    }
}
