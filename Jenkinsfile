
pipeline {
    agent any
 
    parameters {
        string(
            name: 'NAME',
            defaultValue: 'DevOps User',
            description: 'Please tell me your name'
        )
        text(
            name: 'DESC',
            defaultValue: 'Pipeline CI/CD Jenkins GitHub',
            description: 'Description du Job'
        )
        booleanParam(
            name: 'SKIP_TEST',
            defaultValue: false,
            description: 'Skip running Tests ?'
        )
        choice(
            name: 'BRANCH',
            choices: [
                'main',
                'dev',
                'test'
            ],
            description: 'Choose Git branch'
        )
        password(
            name: 'SONAR_SERVER_PWD',
            description: 'Enter SONAR password'
        )
    }
 
    environment {
        APP_NAME    = "web-app"
        DOCKER_IMAGE = "mydockerhub/web-app"
        SONAR_URL   = "http://localhost:9000"
        // Snyk token stocké dans Jenkins Credentials (type "Secret text", id: snyk-token)
        // SNYK_TOKEN  = credentials('snyk-token')  // Décommenter quand le credential est créé
    }
 
    stages {
 
        stage('01 - PRINT PARAMETERS') {
            steps {
                echo "Hello ${params.NAME}"
                echo """
                Job Description : ${params.DESC}
                Branch Selected : ${params.BRANCH}
                Skip Tests      : ${params.SKIP_TEST}
                """
            }
        }
 
        stage('02 - CHECKOUT GITHUB') {
            steps {
                echo "Downloading source code..."
                git branch: "${params.BRANCH}",
                    credentialsId: 'github-token',   // Jenkins Credential (type: Username/Password ou SSH)
                    url: 'https://github.com/Azog1812/web-app.git'
            }
        }
 
        stage('03 - BUILD APPLICATION') {
            tools {
                nodejs 'NodeJS-20'
            }
            steps {
                echo "Installing dependencies..."
                sh 'npm install'
            }
        }
 
        /*
        Scan de sécurité Snyk sur les dépendances npm.
        --severity-threshold=high : le pipeline échoue uniquement
        sur les vulnérabilités HIGH ou CRITICAL.
        Retire ce flag si tu veux échouer sur MEDIUM aussi.
        */
        // stage('04 - SNYK SECURITY SCAN') {
        //     steps {
        //         echo "Running Snyk security scan..."
        //         sh '''
        //             snyk auth $SNYK_TOKEN
        //             snyk test --severity-threshold=high --json-file-output=snyk-report.json || true
        //             snyk monitor
        //         '''
        //     }
        //     post {
        //         always {
        //             archiveArtifacts artifacts: 'snyk-report.json', allowEmptyArchive: true
        //         }
        //     }
        // }
 
        stage('05 - RUN TESTS') {
            tools {
                nodejs 'NodeJS-20'
            }
            when {
                expression { return params.SKIP_TEST == false }
            }
            steps {
                echo "Running unit tests..."
                sh 'npm test'
            }
        }
 
        // stage('06 - SONARQUBE ANALYSIS') {
        //     steps {
        //         echo "Scanning source code with SonarQube..."
        //         sh """
        //         sonar-scanner \\
        //           -Dsonar.projectKey=${APP_NAME} \\
        //           -Dsonar.sources=. \\
        //           -Dsonar.host.url=${SONAR_URL} \\
        //           -Dsonar.login=${params.SONAR_SERVER_PWD}
        //         """
        //     }
        // }
 
        stage('07 - DOCKER BUILD') {
            steps {
                echo "Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE}:latest ."
            }
        }
 
        stage('08 - DEPLOY') {
            steps {
                echo "Deploying application..."
                sh """
                docker stop ${APP_NAME} || true
                docker rm   ${APP_NAME} || true
                docker run -d \\
                    --name ${APP_NAME} \\
                    -p 3000:3000 \\
                    ${DOCKER_IMAGE}:latest
                """
            }
        }
    }
 
    post {
        success {
            echo """
            =========================
            PIPELINE SUCCESS
            Application deployed on port 3000
            =========================
            """
        }
        failure {
            echo """
            =========================
            PIPELINE FAILED
            Please check logs
            =========================
            """
        }
    }
}