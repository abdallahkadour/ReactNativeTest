pipeline {
    agent any

    environment {
       
    }
    
    
    stages {
       

        stage('Install React Native Packages') {
            steps {
                    sh "rm -rf node_modules/"
                    sh "rm -rf package-lock.json && rm -rf yarn.lock"
                    sh "npm install"
            }
        }
        
        stage('Install CocoaPods') {
            steps {
                sh "cd ios && rm -rf Pods && rm -rf Podfile.lock"
                sh "cd ios && export LC_ALL=en_US.UTF-8 && pod install"
            }
        }

        stage('Build APK') {
            steps {
                script {
                    sh "cd android && ./gradlew clean"
                    sh "cd android && ./gradlew assembleRelease"
                }
            }
        }
        
}