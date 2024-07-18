pipeline {
    agent any
    environment {
        def DDESTROY = 'NO'
        def BBUILD = 'NO'
        registry = 'sreeharshav/devopsb20'
        registryCredential = 'dockerhub_id'
        dockerImage = ''
    }
    stages {
        stage('Perform Packer Build') {
            when {
                expression {
                    env.BUILD == 'YES'
                }
            }
            steps {
                    sh 'pwd'
                    sh 'packer build -var-file packer-vars.json packer.json | tee output.txt'
                    sh "tail -2 output.txt | head -2 | awk 'match(\$0, /ami-.*/) { print substr(\$0, RSTART, RLENGTH) }' > ami.txt"
                    sh "echo \$(cat ami.txt) > ami.txt"
                    script {
                        def AMIID = readFile('ami.txt').trim()
                        sh "echo variable \\\"imagename\\\" { default = \\\"$AMIID\\\" } >> variables.tf"
                    }
            }
        }
        stage('Declare AMI') {
            when {
                expression {
                    env.BUILD == 'NO'
                }
            }
            steps {
                    script {
                        def AMIID = 'ami-02ae6bfab37b90006'
                        sh "echo variable \\\"imagename\\\" { default = \\\"$AMIID\\\" } >> variables.tf"
                    }
            }
        }
        stage('Terraform Init & Validate') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform validate'
            }
        }
        stage('Terraform Plan') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform plan'
            }
        }
        stage('Terraform Apply') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform apply --auto-approve'
                    sh 'terraform state list'
            }
        }
        stage('Building Docker Image') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Push Docker image') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Deploy Docker Container') {
            when {
                expression {
                    env.DESTROY == 'NO'
                }
            }
            steps {
                    script {
                    sh 'sleep 30'    
                    sh 'publicip=$(terraform output --raw publicip)'
                    sh 'docker -H $(terraform output --raw publicip):2375 ps'
                    sh 'docker -H $(terraform output --raw publicip):2375 stop nginx01 || docker -H $(terraform output --raw publicip):2375 ps'
                    sh 'docker -H $(terraform output --raw publicip):2375 run --rm -dit --name nginx01 -p 8000:80 sreeharshav/devopsb20:$BUILD_NUMBER'
                    sh 'curl http://$(terraform output --raw publicip):8000'
                    }
            }
        }
        stage('Terraform Destroy') {
            when {
                expression {
                    env.DESTROY == 'YES'
                }
            }
            steps {
                    sh 'terraform init'
                    sh 'terraform destroy --auto-approve'
            }
        }
    }
}
