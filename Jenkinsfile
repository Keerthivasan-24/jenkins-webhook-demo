pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timeout(time: 10, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout & Package') {
            steps {
                sh '''
                    rm -rf dist verified-site
                    mkdir -p dist
                    tar -czf dist/website.tar.gz index.html
                    cd dist && sha256sum website.tar.gz > website.sha256
                    echo "Website package created successfully."
                    ls -lh
                '''
            }
        }

        stage('Verify Package & Content') {
            steps {
                sh '''
                    cd dist && sha256sum -c website.sha256
                    cd ..
                    mkdir -p verified-site
                    tar -xzf dist/website.tar.gz -C verified-site
                    test -s verified-site/index.html
                    grep -q "Jenkins Webhook CI/CD Project" verified-site/index.html
                    grep -q "Created by Keerthivasan" verified-site/index.html

                    printf "Result: PASS\nTriggered by: GitHub Webhook\nBuild number: %s\nPackage checksum: PASS\nWebpage content checks: PASS\n" "$BUILD_NUMBER" > dist/verification-report.txt
                    cat dist/verification-report.txt
                '''
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: 'dist/*', fingerprint: true
            echo 'SUCCESS: Webhook triggered build, package and content verified.'
        }
        failure {
            echo 'FAILED: Open Console Output and inspect the first failing command.'
        }
    }
}
