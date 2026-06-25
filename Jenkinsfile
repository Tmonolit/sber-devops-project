pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
              containers:
              - name: kaniko
                image: gcr.io/kaniko-project/executor:debug
                command:
                - sleep
                args:
                - 9999999
              - name: kubectl
                image: dtzar/helm-kubectl:latest
                command:
                - sleep
                args:
                - 9999999
            '''
        }
    }
    
    environment {
        REGISTRY_ID = 'crpch0cjeu3o3a0vqps4'
        // 1. Поменяли название образа на myproject-app
        IMAGE_NAME  = "cr.yandex/${REGISTRY_ID}/myproject-app:${env.BUILD_NUMBER}"
    }
    
    triggers {
        pollSCM('* * * * *')
    }
    
    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Push (Kaniko)') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'yandex-registry-cred', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        sh '''
                        mkdir -p /kaniko/.docker
                        AUTH=$(echo -n "$USER:$PASS" | base64 | tr -d '\n' | tr -d '\r')
                        echo '{"auths": {"cr.yandex": {"auth": "'$AUTH'"}}}' > /kaniko/.docker/config.json
                        /kaniko/executor --context `pwd` --dockerfile `pwd`/Dockerfile --destination $IMAGE_NAME
                        '''
                    }
                }
            }
        }
        
        stage('K8s Deployment') {
            steps {
                container('kubectl') {
                    sh '''
                    cat <<EOF > deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  // 2. Поменяли имя деплоймента
  name: myproject-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      // 3. Поменяли селекторы и метки
      app: myproject-web-app
  template:
    metadata:
      labels:
        app: myproject-web-app
    spec:
      containers:
      - name: web-app
        image: ${IMAGE_NAME}
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: ycr-secret
EOF
                    // 4. Указываем деплоить в новый неймсейс myproject-app
                    kubectl apply -n myproject-app -f deploy.yaml
                    '''
                }
            }
        }
    }
}
