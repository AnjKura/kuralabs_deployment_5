pipeline {
    agent any
        stages {
            stage ('Build'){
                steps {
                    sh '''#!/bin/bash
                    python3 -m venv test3
                    source  test3/bin/activate
                    pip install pip --upgrade
                    pip install -r requirements.txt
                    export FLASK_APP=application
                    flask run &
                    '''
                }
            }
            
            stage ('Test'){
                steps {
                    sh '''#!/bin/bash
                    source test3/bin/activate
                    py.test --verbose --junit-xml  test-reports/results.xml
                    '''
                }
                post {
                    always {
                        junit 'test-reports/results.xml'
                    }
                }
            }

            stage ('Create Container'){
                agent{label 'Docker-agent'}
                steps {
                withCredentials([string(credentialsId: 'dockerusername', variable: 'dockerusername'),
                                    string(credentialsId: 'dockerpassword', variable: 'dockerpassword')]) {
                    sh '''#!/bin/bash
                    sudo curl https://github.com/AnjKura/kuralabs_deployment_5.git  > dockerfile
                    sudo docker login --username=${dockerusername} --password=${dockerpasssword}
                    sudo docker build -t anjpkura/url_shortener:latest .
                    sudo docker push anjpkura/url_shortener:latest
                    '''
                    }
                }
            }
           stage ('Push to Dockerhub'){
                agent{label 'Docker-agent'}
                steps {
                withCredentials([string(credentialsId: 'dockerusername', variable: 'dockerusername'),
                                    string(credentialsId: 'dockerpassword', variable: 'dockerpassword')]) {
                    sh '''#!/bin/bash
                    sudo curl https://github.com/AnjKura/kuralabs_deployment_5.git  > dockerfile
                    sudo docker login --username=${dockerusername} --password=${dockerpasssword}
                    sudo docker push anjpkura/url_shortener:latest
                    '''
                    }
                }
            }
            stage ('Terraform_Init'){
                agent{label 'terraform_agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform init' 
                                        }
                    }
                }
            }

            stage ('Terraform_Plan'){
                agent{label 'Terraform-Agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform plan' 
                                        }
                    }
                }
            }
            
            stage ('Terraform_Apply'){
                agent{label 'Terraform-Agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform apply' 
                                        }
                    }
                }
            }
             stage ('Deploy to ECS'){
                agent{label 'Terraform-Agent'}
                steps {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY', variable: 'aws_access_key'),
                                    string(credentialsId: 'AWS_SECRET_KEY', variable: 'aws_secret_key')]) {
                                        dir('intTerraform') {
                                        sh 'terraform deploy' 
                                        }
                   }
                }
            }

    }
}
