 pipeline{
    agent {label 'jenkins-Agent'}// المكام ال هتشتغل عليه
    tools{ //البرامج اللي Jenkins بيحتاجها علشان ينفذ المهام.
        jdk 'java21'
        maven 'Maven3'
    }
    environment { //الجزء ده اسمه Environment Variables في Jenkinsfile، ووظيفته إنك تعرّف متغيرات تستخدمها في مراحل الـ Pipeline بدل ما تكرر نفس القيم في كل Stage.
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "mohamed3047205"
        DOCKER_PASS = credentials('dockerhub')
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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
        stage("Quality Gate"){//وظيفتها انتظار نتيجة فحص الكود من SonarQube وتحديد هل الـ Pipeline تكمل ولا تتوقف
            steps{
                script{
                    waitForQualityGate abortPipeline: false,credentialsId: 'jenkins-sonarqube-token'
                }
            }
        }
        stage("Build & push Docker Image"){
            steps{
                script{
                    docker.withRegistry('', 'dockerhub') {//بيعمل Authentication مع Docker Hub باستخدام الـ Credentials المخزنة في Jenkins.
                       docker_image = docker.build("${IMAGE_NAME}")//بيقرأ الـ Dockerfile ويعمل Build للـ Image ويحفظ الـ Reference بتاعها في متغير
                        docker_image.push("${IMAGE_TAG}")//بيرفع الـ Image على Docker Hub برقم الإصدار ورقم الـ Build.
                        docker_image.push('latest')//يرفع نفس الـ Image باستخدام Tag اسمه latest للإشارة للإصدار الأحدث اللي رفعته.
                    }
                }
            }
        }
        stage("Trivy Scan") {//ووظيفتها فحص الـ Docker Image واكتشاف الثغرات الأمنية الموجودة في الـ Packages والـ Dependencies ونظام التشغيل الموجود داخل الـ Image
            steps {
                script {
                    sh ('docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image mohamed3047205/register-app-pipeline:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table')
                }
            }
        }
        stage ('Cleanup Artifacts') {
            steps {
               script {
                   sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
           }
       }    
    }




}