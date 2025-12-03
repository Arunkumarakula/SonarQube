pipeline {
    agent any

    environment {
        DOCKER_IMAGE      = "scheduling-service-test"
        DOCKER_TAG        = "latest"
        CONTAINER_NAME    = "scheduling-service-test"
        SONAR_PROJECT_KEY = "scheduling-service"
        SONAR_HOST_URL    = "http://3.7.48.8:9000"
        SONAR_REPORT_DIR  = "${WORKSPACE}/sonar_reports"
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git branch: 'Develop',
                    url: 'https://github.com/arunkumarakula/mfit-be-scheduling-service.git',
                    credentialsId: 'my-token'
            }
        }

        stage('Build Artifact') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner' 
                    withSonarQubeEnv('SonarQube') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=$SONAR_PROJECT_KEY -Dsonar.sources=src -Dsonar.java.binaries=target/classes -Dsonar.host.url=$SONAR_HOST_URL"
                    }
                }
            }
        }

        stage('Fetch Sonar Reports') {
            steps {
                withCredentials([string(credentialsId: 'Jenkins-Sonar-Token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        mkdir -p "$SONAR_REPORT_DIR"

                        # Issues report in boxed format
                        curl -s -u $SONAR_TOKEN: "$SONAR_HOST_URL/api/issues/search?projectKeys=$SONAR_PROJECT_KEY&ps=500" \
                        | jq -r '.issues[]? | "----------------------------------------\\nSeverity: "+.severity+"\\nType: "+.type+"\\nComponent: "+.component+"\\nLine: "+(.line|tostring)+"\\nMessage: "+.message+"\\n----------------------------------------"' \
                        > "$SONAR_REPORT_DIR/issues-report-$BUILD_ID.txt" || true

                        # Measures report as TXT
                        curl -s -u $SONAR_TOKEN: "$SONAR_HOST_URL/api/measures/component?component=$SONAR_PROJECT_KEY&metricKeys=coverage,bugs,vulnerabilities,code_smells,duplicated_lines_density,ncloc,complexity" \
                        | jq -r '.component.measures[]? | .metric + ": " + (.value // "N/A")' \
                        > "$SONAR_REPORT_DIR/measures-report-$BUILD_ID.txt" || true

                        # Quality Gate report as TXT
                        curl -s -u $SONAR_TOKEN: "$SONAR_HOST_URL/api/qualitygates/project_status?projectKey=$SONAR_PROJECT_KEY" \
                        | jq -r '.projectStatus | "Status: "+.status + "\\n" + (.conditions[]? | .metricKey + ": " + .status + " (value=" + (.actualValue|tostring) + ")")' \
                        > "$SONAR_REPORT_DIR/quality-gate-report-$BUILD_ID.txt" || true

                        echo "SonarQube reports saved in $SONAR_REPORT_DIR"
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker rm -f $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME -p 8013:8080 $DOCKER_IMAGE:$DOCKER_TAG
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'sonar_reports/*.txt', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. SonarQube reports archived under $SONAR_REPORT_DIR/"
        }
    }
}
