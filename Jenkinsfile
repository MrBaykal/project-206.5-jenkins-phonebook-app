pipeline {
    agent any
    environment {
        APP_NAME="phonebook"
        APP_REPO_NAME="baykal/${APP_NAME}-app"
        AWS_ACCOUNT_ID=sh(script:'aws sts get-caller-identity --query Account --output text', returnStdout:true).trim()
        AWS_REGION="us-east-1"
        ECR_REGISTRY="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        PATH="$PATH:/tmp:/var/lib/jenkins/workspace/phonebook-jenkins"
        }
        stages {
            stage('Create ECR Repo') {
                steps {
                    echo "Creating ECR Repo for ${APP_NAME}-app"
                    sh '''
                    aws ecr describe-repositories --region ${AWS_REGION} --repository-name ${APP_REPO_NAME} || \
                            aws ecr create-repository \
                            --repository-name ${APP_REPO_NAME} \
                            --image-scanning-configuration scanOnPush=true \
                            --image-tag-mutability MUTABLE \
                            --region ${AWS_REGION}
                    '''
                        }
                    }
                    stage('Build App Docker Images for WEB') {
                steps {
                    echo 'Building App Images for web_server'
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    cd 02_image_files/image_for_web_server
                    docker build -t my_repo/phonebook-app:web .
                    docker tag my_repo/phonebook-app:web ${ECR_REGISTRY}/${APP_REPO_NAME}:web
                    docker push ${ECR_REGISTRY}/${APP_REPO_NAME}:web
                    docker image ls
                    '''
                }
            }
            stage('Build App Docker Images for RESULT') {
                steps {
                    echo 'Building App Images for result_server'
                    sh '''
                    aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    cd 02_image_files/image_for_result_server
                    docker build -t my_repo/phonebook-app:result .
                    docker tag my_repo/phonebook-app:result ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker push ${ECR_REGISTRY}/${APP_REPO_NAME}:result
                    docker image ls
                    '''
                }
            }
            stage('Installing eksctl, kubectl and creating EKS Cluster and deploying Phonebook Application') {
                steps {
                    echo 'Installing eksctl, kubectl and creating EKS Cluster and deploying Phonebook Application'
                    sh '''
                    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
                    eksctl version

                    curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.26.4/2023-05-11/bin/linux/amd64/kubectl
                    chmod +x ./kubectl
                    kubectl version --short --client
                    
                    eksctl create cluster -f 03_cluster/cluster.yaml
                    
                    kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/cloud/deploy.yaml

                    sleep 200

                    cd 04_yaml_files

                    kubectl apply -f .

                    echo "Now we can see our application for 300 seconds from the DNS address of the Load Balancer."

                    sleep 300

                    '''
                    
                }
        }
    }
    post {
        always {
            echo 'Deleting all local images'
            sh 'docker image prune -af'

            echo 'Delete the Image Repository on ECR'
            sh '''
                aws ecr delete-repository \
                  --repository-name ${APP_REPO_NAME} \
                  --region ${AWS_REGION}\
                  --force
                '''

            echo 'Delete the Application and yaml files'            
            sh '''
            cd 04_yaml_files
            pwd
            kubectl delete -f .
            '''

            echo 'Tear down the EKS Cluster'
            sh '''
            cd 03_cluster
            pwd
            eksctl delete cluster -f cluster.yaml
            '''
        }
    }
}
        

// ;         Bu komut, Jenkinsfile'da PATH ortam değişkenini ayarlar. PATH değişkeni, bilgisayarınızda bulunan komutları ve programları bulmak için kullanılan bir yol listesidir. Bu komut, PATH değişkenine /tmp ve /var/lib/jenkins/workspace/phonebook-jenkins dizinlerini ekler.
// ; Bu komutun amacı, Jenkinsfile'da kullanılan komutların ve programların bu dizinlerde bulunmasını sağlamaktır. Örneğin, /tmp dizini, geçici dosyalar için kullanılır. Jenkinsfile'da geçici dosyalar kullanıyorsanız, bu dosyaları /tmp dizininde bulabilmek için bu komutu kullanmanız gerekir.
// ; /var/lib/jenkins/workspace/phonebook-jenkins dizini, Jenkins'in Phonebook projesi için kullandığı çalışma alanıdır. Bu dizine, projenin kaynak kodları, derlenmiş dosyaları ve diğer dosyaları bulunur. Bu komut, Jenkinsfile'da bu dosyaları kullanabilmek için PATH değişkenine bu dizini ekler.
// ; Özetle, bu komut, Jenkinsfile'da kullanılan komutların ve programların /tmp ve /var/lib/jenkins/workspace/phonebook-jenkins dizinlerinde bulunmasını sağlar.
// ; Bu komutu anlamak için biraz daha teknik bilgi vereyim.
// ; PATH değişkeni, bir dizi dizin yolu içeren bir ortam değişkenidir. Bu dizinler, kabuk tarafından komutları ve programları bulmak için kullanılır. Kabuk, bir komutu çalıştırırken, önce PATH değişkeninde listelenen dizinleri kontrol eder. Bu dizinlerde komutu bulursa, komutu çalıştırır.
// ; Bu komut, PATH değişkenine yeni dizin yolları ekleyerek kabuğun komutları ve programları bulabileceği dizinleri genişletir.
// ; Bu komutu, Jenkinsfile'da aşağıdaki gibi kullanabilirsiniz:
// ; def setup() {
// ;   PATH="$PATH:/tmp:/var/lib/jenkins/workspace/phonebook-jenkins"
// ; }
// ; Bu kod, setup() fonksiyonunu tanımlar. setup() fonksiyonu, Jenkinsfile'ın yürütülmesi sırasında bir kez çalıştırılır. Bu fonksiyonda, PATH değişkenine yeni dizin yolları eklenir.
// ; Bu komutu kullanarak, Jenkinsfile'da daha fazla komut ve program kullanmaya başlayabilirsiniz.