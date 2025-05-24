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
                        git url: 'https://github.com/stanoz/tdo_lab11', branch: 'main'
                    }
         }

        stage('Validation') {
            steps {
                           sh '''
                               [ -d demo_tdo_lab11/src/main/java ] || { echo "Brakuje katalogu demo_tdo_lab11/src/main/java"; exit 1; }
                               [ -d demo_tdo_lab11/src/test/java ] || { echo "Brakuje katalogu demo_tdo_lab11/src/test/java"; exit 1; }
                               [ -d demo_tdo_lab11/lib ] || { echo "Brakuje katalogu demo_tdo_lab11/lib/"; exit 1; }
                           '''
                       }
        }

        stage('Build') {
            steps {
                sh '''
                    mkdir -p ${CLASS_DIR} ${TEST_DIR} ${REPORT_DIR}

                    javac -cp "demo_tdo_lab11/lib/*" -d ${CLASS_DIR} $(find demo_tdo_lab11/src/main/java -name "*.java")
                    javac -cp "${CLASS_DIR}:demo_tdo_lab11/lib/*" -d ${TEST_DIR} $(find demo_tdo_lab11/src/test/java -name "*.java")
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
                            java -jar demo_tdo_lab11/lib/junit-platform-console-standalone.jar \
                              --class-path target/classes:target/test-classes:demo/lib/* \
                              --scan-class-path \
                              --reports-dir=target/reports
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
