pipeline {
  agent any
  environment {
    AWS_REGION = 'us-east-1'
    S3_BUCKET  = 'slean-cft-artifacts-bucket'
    STACK_NAME = 'my-stack'
    TEMPLATE   = 'template.yaml'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Debug') {
      steps {
        sh '''
          echo "PWD inside pipeline: $(pwd)"
          echo "Listing contents:"
          ls -al
          aws sts get-caller-identity
        '''
      }
    }

    stage('Validate') {
      steps {
        sh """
          aws cloudformation validate-template \
            --region ${env.AWS_REGION} \
            --template-body file://${env.TEMPLATE} || true
        """
      }
    }

    stage('Package (if needed)') {
      steps {
        sh """
          if grep -q "CodeUri\\|Content\\|ZipFile" ${env.TEMPLATE}; then
            echo "Packaging template because it references local artifacts..."
            aws cloudformation package \
              --region ${env.AWS_REGION} \
              --template-file ${env.TEMPLATE} \
              --s3-bucket ${env.S3_BUCKET} \
              --output-template-file packaged.yaml
          else
            echo "No local artifacts detected, skipping package step"
            cp ${env.TEMPLATE} packaged.yaml
          fi
        """
      }
    }

    stage('Deploy') {
      steps {
        sh """
          aws cloudformation deploy \
            --region ${env.AWS_REGION} \
            --template-file packaged.yaml \
            --stack-name ${env.STACK_NAME} \
            --capabilities CAPABILITY_NAMED_IAM CAPABILITY_IAM \
            --parameter-overrides KeyName=Aadvik \
            --no-fail-on-empty-changeset
        """
      }
    }
  }

  post {
    success {
      echo "✅ CloudFormation deploy succeeded!"
    }
    failure {
      echo "❌ CloudFormation deploy failed"
      archiveArtifacts artifacts: '**/packaged.yaml', allowEmptyArchive: true
    }
  }
}
