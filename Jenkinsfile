pipeline {
    agent any

    options {
        timestamps()
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/priyamuthu062105-lang/HabitApp.git'
            }
        }

        stage('Maven Build and Test') {
            steps {
                sh '''
                    java -version
                    mvn -version
                    mvn clean verify
                '''
            }
        }

        stage('SonarQube Cloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=priyamuthu062105-lang_HabitApp \
                          -Dsonar.organization=priyamuthu062105-lang \
                          -Dsonar.host.url=https://sonarcloud.io
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t habitapp:${BUILD_NUMBER} .
                    docker images habitapp
                '''
            }
        }

        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" --password-stdin

                        docker tag habitapp:${BUILD_NUMBER} \
                            $DOCKER_USER/habitapp:${BUILD_NUMBER}

                        docker push $DOCKER_USER/habitapp:${BUILD_NUMBER}

                        docker logout
                    '''
                }
            }
        }

        stage('Helm Validation') {
            steps {
                sh 'helm lint ./habitapp-chart'
            }
        }

        stage('GitOps Update') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'github-token',
                        variable: 'GITHUB_TOKEN'
                    ),
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        set -eu

                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@example.com"

                        sed -i "s|^[[:space:]]*repository:.*|  repository: ${DOCKER_USER}/habitapp|" habitapp-chart/values.yaml

                        sed -i "s|^[[:space:]]*tag:.*|  tag: \\"${BUILD_NUMBER}\\"|" habitapp-chart/values.yaml

                        echo "Updated Helm image configuration:"
                        grep -E 'repository:|tag:' habitapp-chart/values.yaml

                        git add habitapp-chart/values.yaml
                        git diff --cached --check

                        if ! git diff --cached --quiet; then

                            git commit -m "Update HabitApp image to ${BUILD_NUMBER}"

                            ASKPASS=$(mktemp)
                            trap 'rm -f "$ASKPASS"' EXIT

                            cat > "$ASKPASS" <<'EOF'
#!/bin/sh
case "$1" in
    *Username*) echo "x-access-token" ;;
    *Password*) echo "$GITHUB_TOKEN" ;;
esac
EOF

                            chmod 700 "$ASKPASS"

                            GIT_ASKPASS="$ASKPASS" \
                            GIT_TERMINAL_PROMPT=0 \
                            git push origin HEAD:main

                        else
                            echo "No Helm changes to push."
                        fi
                    '''
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true,
                  testResults: 'target/surefire-reports/*.xml'

            archiveArtifacts artifacts: 'target/*.jar',
                             allowEmptyArchive: true
        }

        success {
            echo 'HabitApp CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'HabitApp CI/CD Pipeline failed. Check the console output.'
        }
    }
}
