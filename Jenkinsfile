 pipeline{
    agent {label 'jenkins-Agent'}// المكام ال هتشتغل عليه
    tools{ //البرامج اللي Jenkins بيحتاجها علشان ينفذ المهام.
        jdk 'java21'
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
        stage("build application"){ //يبني البرنامج
            steps{
                sh "mvn clean package -DskipTests"
            }
        }
        stage("Test Application"){ // بيجربه
            steps{
                sh "mvn test"
            }
        }
        stage("SonarQube Analysis"){ // كود تشغيل السوناركيوب 
            steps{
                script{
                    withSonarQubeEnv(credentialsId:'jenkins-sonarqube-token'){
                        sh "mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar"
                    }
                }
            }
        }
        stage("Quality Gate"){
            steps{
                script{
                    waitForQualityGate abortPipeline: false,credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }

    }




}