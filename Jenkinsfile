pipeline {
  agent any

  environment {
    JAVA_HOME = '/opt/java/openjdk'
    ANDROID_HOME = '/opt/android-sdk'
    PATH = "${JAVA_HOME}/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/cmdline-tools/latest/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    NODE_OPTIONS = '--max_old_space_size=4096'
  }

    
    stages {
        stage('Check Java Version') {
            steps {
                script {
                    def javaVersion = sh(script: 'java -version 2>&1 | head -n 1', returnStdout: true).trim()
                    echo "✔ Java version ${javaVersion} is OK."
                }
            }
        }
        
        stage('Check Node Version') {
            steps {
                script {
                    String nodeVersion = sh(script: 'node -v', returnStdout: true).trim()
                    echo "✔ Node.js version ${nodeVersion} is OK."
                }
            }
        }
        
        stage('Check Gradle Version') {
            steps {
                echo 'Gradle version (using wrapper)...'
                dir('android') {
                    sh 'chmod +x ./gradlew'
                    sh './gradlew --version'
                }
            }
        }
        
        stage('Checkout SCM') {
            steps {
                echo 'Cloning the source repository...'
                git branch: 'main', url: 'https://github.com/abdallahkadour/ReactNativeTest'
            }
        }
        
        stage('Install dependencies') {
            steps {
                echo 'Cleaning and installing dependencies...'
                sh 'rm -rf node_modules package-lock.json'
                sh 'npm install --legacy-peer-deps'
                
                echo 'Verifying installations...'
                sh '''
                    ls -la node_modules/react-native || echo "React Native not found!"
                    ls -la node_modules/@react-native/gradle-plugin || echo "Gradle plugin not found!"
                '''
            }
        }
        
        stage('Clean Build') {
            steps {
                echo 'Cleaning build directories...'
                dir('android') {
                    sh 'chmod +x ./gradlew'
                    sh 'rm -rf .gradle build app/build'
                }
                sh 'rm -rf /tmp/metro-* /tmp/haste-* /tmp/react-native-packager-cache-*'
            }
        }
        
        stage('Build Android APK') {
            steps {
                dir('android') {
                    echo 'Building Android APK...'
                    sh './gradlew assembleDebug --no-daemon --stacktrace'
                }
            }
        }
        
        stage('Archive APK') {
            steps {
                echo 'Archiving APK...'
                archiveArtifacts artifacts: 'android/app/build/outputs/apk/debug/app-debug.apk', fingerprint: true
            }
        }
    }
    
    post {
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Error: Android build failed!'
        }
    }
}