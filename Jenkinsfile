 pipeline{
    agent {label 'jenkins-Agent(docker)'}// المكام ال هتشتغل عليه
    tools{ //البرامج اللي Jenkins بيحتاجها علشان ينفذ المهام.
        jdk 'java17'
        maven 'Maven3'
    }
    stages{ //مراحل التنفيذ 
        stage ("Cleanup Workspace"){//يمسح الحاجات القديمه

            steps{
                cleanWs()
            }
        }   
        stage("Checkout from SCM"){// يسحب الكود من github 
            steps{
                git branch:'main', credentialsId:'github', url: 'https://github.com/Mowalied1/register-app.git'
            }
        }
        stage("build application"){
            steps{
                sh "mvn clean package"
            }
        }
        stage("Test Application"){
            steps{
                sh "mvn test"
            }
        }

    }




}