pipeline {
  agent any

  parameters {
    choice(name: 'ENVIRONMENT', choices: ['dev'], description: 'Target environment for Terraform')
    choice(name: 'ACTION', choices: ['plan', 'apply', 'destroy'], description: 'Terraform action to execute')
    booleanParam(name: 'AUTO_APPROVE', defaultValue: false, description: 'Auto-approve Terraform apply/destroy actions')
  }

  environment {
    TF_IN_AUTOMATION = 'true'
    TF_DIR = 'terraform/envs/dev'
    TF_BACKEND_CONFIG = 'terraform/backend/dev.hcl'
  }

  stages {
    stage('Checkout Source Code') {
      steps {
        checkout scm
      }
    }

    stage('Terraform Version') {
      steps {
        sh 'terraform version'
      }
    }

    stage('Terraform Format Check') {
      steps {
        sh 'terraform fmt -check -recursive terraform'
      }
    }

    stage('Terraform Init') {
      steps {
        dir(env.TF_DIR) {
          sh "terraform init -backend-config=../../backend/dev.hcl -input=false"
        }
      }
    }

    stage('Terraform Validate') {
      steps {
        dir(env.TF_DIR) {
          sh 'terraform validate'
        }
      }
    }

    stage('Terraform Plan') {
      steps {
        dir(env.TF_DIR) {
          sh 'terraform plan -var-file=dev.tfvars -out=tfplan'
        }
      }
    }

    stage('Manual Approval') {
      when {
        expression {
          (params.ACTION == 'apply' || params.ACTION == 'destroy') && params.AUTO_APPROVE != true
        }
      }
      steps {
        input message: 'Approve Terraform execution', ok: 'Proceed'
      }
    }

    stage('Terraform Apply') {
      when {
        expression {
          params.ACTION == 'apply'
        }
      }
      steps {
        dir(env.TF_DIR) {
          sh 'terraform apply -auto-approve -input=false tfplan'
        }
      }
    }

    stage('Terraform Destroy') {
      when {
        expression {
          params.ACTION == 'destroy'
        }
      }
      steps {
        dir(env.TF_DIR) {
          sh 'terraform destroy -var-file=dev.tfvars -auto-approve'
        }
      }
    }
  }
}
