node {

    def mavenHome
    def mavenCMD
    def tagName

    stage('Prepare Environment') {
        echo 'Initializing variables...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
        tagName = "v1.0"
    }

    stage('Git Checkout') {
        try {
            echo 'Cloning Git repository...'
            git branch: 'master', url: 'https://github.com/Shriraksha384/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo '❌ Error during Git Checkout'
            currentBuild.result = "FAILURE"
            error("Stopping the Pipeline due to Git failure")
        }
    }

   stage('Build the Application') {
    echo "Cleaning... Compiling... Packaging..."
    // Run Maven from ROOT (where pom.xml exists)
    sh "${mavenCMD} clean package -DskipTests"
}
    

    stage('Push to AWS ECR') {
        echo 'Logging in to AWS ECR & Pushing Docker Image...'
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
            sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 399485458177.dkr.ecr.ap-south-1.amazonaws.com
                docker tag insure-me:${tagName} 399485458177.dkr.ecr.ap-south-1.amazonaws.com/insureme:${tagName}
                docker push 399485458177.dkr.ecr.ap-south-1.amazonaws.com/insureme:${tagName}
            """
        }
    }

    stage('Deploy to Test Server (8081)') {
        echo "Deploying container on Test server..."
        sh """
            docker rm -f insureme-test || true
            docker run -d --name insureme-test -p 8081:8080 399485458177.dkr.ecr.ap-south-1.amazonaws.com/insureme:${tagName}
        """
    }stage('Push Docker Image') {
    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
        sh "echo $PASS | docker login -u $USER --password-stdin"
        sh "docker tag insureme:1.0.0 yourdockerhubusername/insureme:1.0.0"
        sh "docker push yourdockerhubusername/insureme:1.0.0"
    }
}


    stage('Approval for Production Deployment?') {
        timeout(time: 30, unit: 'MINUTES') {
            input message: 'Deploy to Production?', ok: 'Yes, Deploy'
        }
    }

    stage('Deploy to Production Server (8082)') {
        echo "Deploying container on Production..."
        sh """
            docker rm -f insureme-prod || true
            docker run -d --name insureme-prod -p 8082:8080 399485458177.dkr.ecr.ap-south-1.amazonaws.com/insureme:${tagName}
        """
    }
}
