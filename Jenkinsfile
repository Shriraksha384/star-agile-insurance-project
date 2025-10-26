node {

    def mavenHome
    def mavenCMD
    def tagName = "1.0.0"   // ✅ Use consistent tag everywhere
    def ecrRepo = "399485458177.dkr.ecr.ap-south-1.amazonaws.com/insureme"

    stage('📂 Prepare Environment') {
        echo 'Initializing variables...'
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD = "${mavenHome}/bin/mvn"
    }

    stage('🌱 Git Checkout') {
        try {
            echo 'Cloning Git repository...'
            git branch: 'master', url: 'https://github.com/Shriraksha384/star-agile-insurance-project.git'
        } catch (Exception e) {
            echo '❌ Error during Git Checkout'
            currentBuild.result = "FAILURE"
            error("Stopping the Pipeline due to Git failure")
        }
    }

    stage('⚙️ Build the Application') {
        echo "Cleaning... Compiling... Packaging..."
        sh "${mavenCMD} clean package -DskipTests"
    }

    stage('🐳 Build Docker Image') {
        sh """
            docker build -t insureme:${tagName} .
        """
    }

    stage('🚀 Push to AWS ECR') {
        echo "Pushing Docker Image to AWS ECR..."
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
            sh """
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin ${ecrRepo}
                docker tag insureme:${tagName} ${ecrRepo}:${tagName}
                docker push ${ecrRepo}:${tagName}
            """
        }
    }

    stage('🧪 Deploy to Test Server (8081)') {
        echo "Deploying container on Test Server:8081..."
        sh """
            docker rm -f insureme-test || true
            docker run -d --name insureme-test -p 8081:8080 ${ecrRepo}:${tagName}
        """
    }

    stage('📤 Push Docker Image to DockerHub') {
        withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
            sh """
                echo $PASS | docker login -u $USER --password-stdin
                docker tag insureme:${tagName} $USER/insureme:${tagName}
                docker push $USER/insureme:${tagName}
            """
        }
    }

    stage('✅ Approval for Production Deployment?') {
        timeout(time: 20, unit: 'MINUTES') {
            input message: 'Deploy to Production?', ok: 'Yes, Deploy'
        }
    }

    stage('🚢 Deploy to Production Server (8082)') {
        echo "Deploying container on Production Server:8082..."
        sh """
            docker rm -f insureme-prod || true
            docker run -d --name insureme-prod -p 8082:8080 ${ecrRepo}:${tagName}
        """
    }
}
