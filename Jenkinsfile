// app-repo/Jenkinsfile
pipeline {
    agent any

    environment {
        // ######### 需要您修改的变量 #########
        DOCKER_IMAGE_NAME = "jin00000/app-repo-image" // 1. 替换为您的 Docker Hub 用户名和镜像名
        CONFIG_REPO_URL = "https://github.com/jin00100/config-repo.git" // 2. 替换为您的配置仓库地址
        // ###################################

        GIT_CREDS_ID = 'github-creds'      // GitHub 凭证 ID
        DOCKER_CREDS_ID = 'dockerhub-creds'  // DockerHub 凭证 ID
    }

    stages {
        stage('1. Build and Push Image') {
            steps {
                script {
                    def imageWithTag = "${DOCKER_IMAGE_NAME}:${env.BUILD_ID}"
                    echo "Building image: ${imageWithTag}"
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDS_ID) {
                        def img = docker.build(imageWithTag, '.')
                        img.push()
                    }
                }
            }
        }

        stage('2. Update Staging Config Repo') {
            steps {
                script {
                    def imageWithTag = "${DOCKER_IMAGE_NAME}:${env.BUILD_ID}"

                    // 使用 Git 插件克隆配置仓库，并自动注入凭证
                    git credentialsId: GIT_CREDS_ID, url: CONFIG_REPO_URL

                    sh '''
                        echo "Updating staging config to use image ${imageWithTag}"

                        # 配置 git 提交者信息
                        git config user.email "jenkins@example.com"
                        git config user.name "Jenkins Bot"

                        # 使用 Kustomize 更新 staging 环境的镜像
                        # 这个命令会修改 overlays/staging/kustomization.yaml 文件
                        cd overlays/staging
                        kustomize edit set image ${DOCKER_IMAGE_NAME}=${imageWithTag}
                        cd ../..

                        # 提交并推送变更到 config-repo
                        git add .
                        git commit -m "Update image to ${imageWithTag} for staging [CI Build ${env.BUILD_ID}]"
                        git push origin HEAD:main
                    '''
                }
            }
        }
    }
}
