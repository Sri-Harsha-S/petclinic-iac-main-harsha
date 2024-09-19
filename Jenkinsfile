pipeline {
    agent any
    parameters {
        choice(name: 'ACTION', choices: ['apply', 'destroy'], description: 'Choose the Terraform action to perform')
    }
    environment {
        APP_REPO_NAME_1 = "petclinic/api-gateway"
        APP_REPO_NAME_2 = "petclinic/api-gateway-staging"
        APP_REPO_NAME_3 = "petclinic/customers-service"
        APP_REPO_NAME_4 = "petclinic/customers-service-staging"
        APP_REPO_NAME_5 = "petclinic/vets-service"
        APP_REPO_NAME_6 = "petclinic/vets-service-staging"
        APP_REPO_NAME_7 = "petclinic/visits-service"
        APP_REPO_NAME_8 = "petclinic/visits-service-staging"
        AWS_REGION = "eu-west-3"
        AWS_ACCESS_KEY_ID = credentials('ACCESS_KEY_AWS')
        AWS_SECRET_ACCESS_KEY = credentials('SECRET_KEY_AWS')
        TF_VAR_DB_USERNAME = "${DB_USERNAME}"
        TF_VAR_DB_PASSWORD = "${DB_PASSWORD}"
        TF_VAR_GRAFANA_PASSWORD = "${GRAFANA_PASSWORD}"
    }

    stages {
        stage('Create ECR Repositories') {
            steps {
                script {
                    def repos = [
                        env.APP_REPO_NAME_1,
                        env.APP_REPO_NAME_2,
                        env.APP_REPO_NAME_3,
                        env.APP_REPO_NAME_4,
                        env.APP_REPO_NAME_5,
                        env.APP_REPO_NAME_6,
                        env.APP_REPO_NAME_7,
                        env.APP_REPO_NAME_8
                    ]

                    repos.each { repoName ->
                        sh """
                            aws ecr describe-repositories --region "${env.AWS_REGION}" --repository-name "${repoName}" || \
                            aws ecr create-repository \
                              --repository-name "${repoName}" \
                              --image-scanning-configuration scanOnPush=false \
                              --image-tag-mutability MUTABLE \
                              --region "${env.AWS_REGION}"
                        """
                    }
                }
            }
        }
        stage("Git Checkout") {
            steps {
                git(
                    url: "https://github.com/Sri-Harsha-S/petclinic-iac-main-harsha.git",
                    branch: "personal",
                    credentialsId: 'GITHUB_TOKEN'
                )
            }
        }
        stage("Terraform init") {
            steps {
                script {
                    sh "terraform init -reconfigure"
                    sh "terraform init"
                }
            }
        }
        stage("Terraform apply/destroy") {
            steps {
                script {
                    sh "terraform ${params.ACTION} -var-file='secrets.tfvars' -auto-approve"
                }
            }
        }
        stage("Connecting locally to the EKS cluster") {
            when {
                expression {
                    return params.ACTION == "apply"
                }
            }
            steps {
                script {
                    sh "aws eks update-kubeconfig --name petclinic-eks"
                    //sh "sudo cp /var/lib/jenkins/.kube/config /home/ec2-user/.kube/config"
                    //sh "kubectl create namespace argocd"
                    //sh "kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml"
                    //sh "kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'"
                }
            }

        }
    }
}