pipeline {
  agent any

  environment {
    JAVA_HOME = '/opt/java/openjdk'
    ANDROID_HOME = '/opt/android-sdk'
    PATH = "${JAVA_HOME}/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/cmdline-tools/latest/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
    NODE_OPTIONS = '--max_old_space_size=4096'
    // CRITICAL FIX: Explicitly define the full path to the NDK we confirmed should be installed.
    // This property will be passed to Gradle to bypass the failing NDK installation check.
    NDK_FULL_PATH = '/opt/android-sdk/ndk/27.1.12297006' 
  }

  stages {
    stage('Check Java Version') {
      steps {
        script {
          // Capture first line of java -version output
          String javaVersionOutput = sh(
            returnStdout: true,
            script: 'java -version 2>&1 | head -n 1'
          ).trim()

          // Example: openjdk version "17.0.17"
          // Extract the version inside the quotes
          String versionString = javaVersionOutput.find(/\"([0-9.]+)\"/)
            ?.replace('\"', '')

          // Extract major version (17 from 17.0.17)
          int major = versionString.split('\\.')[0].toInteger()

          if (major < 17) {
            error "❌ Java version ${versionString} is too old. Java 17 or higher is required."
          } else {
            echo "✔ Java version ${versionString} is OK."
          }
        }
      }
    }

    stage('Check Node Version') {
      steps {
        script {
          // Read Node.js version string, e.g., "v18.17.0"
          String nodeVersion = sh(returnStdout: true, script: 'node -v').trim()

          // Extract major version as integer
          int major = nodeVersion.replace('v', '').split('\\.')[0].toInteger()

          if (major < 20) {
            error "❌ Node.js version ${nodeVersion} is too old. Version 20 or higher is required."
          } else {
            echo "✔ Node.js version ${nodeVersion} is OK."
          }
        }
      }
    }

    stage('Checkout SCM') {
      steps {
        sh 'find /var/jenkins_home/tools -name "git" -executable -type f'
        sh "chmod +x /var/jenkins_home/tools"
        echo 'Cloning the source repository...'
        git branch: 'main',
          credentialsId: '25a28d84-e151-45ac-a438-f42b885f8e26',
          url: 'https://dev.azure.com/netways-uae/CST-10237/_git/mojapp'
      }
    }

    
    stage('Check Gradle Version') {
      steps {
        echo 'Gradle version (using wrapper)...'
        sh 'cd android && chmod +x ./gradlew && ./gradlew --version'
      }
    }


    stage('Install dependencies') {
      steps {
        echo 'Cleaning and installing Node modules...'
        sh 'rm -rf node_modules'
        sh 'rm -f package-lock.json'
        sh 'npm cache clean --force'
        sh 'node -v'
        sh 'npm -v'
        sh 'npm install --legacy-peer-deps'
      }
    }

    stage('Build Android APK') {
      steps {
        echo 'Cleaning and building Android Release APK (Forcing NDK Path)...'
        sh """
            cd android
            chmod +x ./gradlew
            
            # We are explicitly setting the NDK path using -Pandroid.ndkPath to bypass the read-only check.
            
            # 1. Clean build artifacts and cache
            ./gradlew clean cleanBuildCache -Pandroid.ndkPath=\${NDK_FULL_PATH}

            # 2. Generate package list (as specified in your original script)
            ./gradlew :app:generatePackageList -Pandroid.ndkPath=\${NDK_FULL_PATH}

            # 3. Final release build (using --stacktrace and --info for verbosity)
            ./gradlew assembleRelease --stacktrace --info -Pandroid.ndkPath=\${NDK_FULL_PATH}
        """
      }
    }

    stage('Archive APK') {
      steps {
        echo 'Archiving release APK...'
        archiveArtifacts artifacts: 'android/app/build/outputs/apk/release/app-release.apk', fingerprint: true
        echo 'APK archived successfully!'
      }
    }
  }

  post {
    success {
      echo 'Android build and deployment completed successfully!'
    }
    failure {
      echo 'Error: Android build failed!'
    }
  }
}