pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'guddnr5345/devops-test'
        // Config 레포지토리 주소 (https 제외, .git 제외 권장)
        CONFIG_REPO_URL = 'github.com/haengguk/devops-test-config.git'
        // Config 레포 내의 매니페스트 파일 이름
        MANIFEST_FILE = 'deployment.yaml'
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/haengguk/devops-test.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // 빌드번호 & latest 태그로 빌드
                    sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        // Config 레포지토리 업데이트
        stage('Update Manifest') {
            steps {
                script {
                    // 아까 만든 GitHub 토큰 ID (cicd-token) 사용
                    withCredentials([usernamePassword(credentialsId: 'cicd-token', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {

                        // 1. Config 레포지토리 클론 (임시 폴더 'config-repo'에 다운로드), 인증 정보를 URL에 포함시켜서 클론
                        sh "git clone https://${GIT_USER}:${GIT_TOKEN}@${CONFIG_REPO_URL} config-repo"

                        // 2. 폴더 안으로 이동해서 내용 수정
                        dir('config-repo') {
                            // Git 커밋을 위한 사용자 설정 (필수)
                            sh "git config user.email 'flint@flint'"
                            sh "git config user.name 'Jenkins-Bot'"

                            // 3. sed 명령어로 deployment.yaml 파일의 이미지 태그 부분 수정
                            // 예: image: guddnr5345/devops-test:10 -> image: guddnr5345/devops-test:11
                            sh "sed -i 's|image: ${DOCKER_IMAGE}:.*|image: ${DOCKER_IMAGE}:${BUILD_NUMBER}|' ${MANIFEST_FILE}"

                            // 4. 변경사항 확인 (로그용)
                            sh "cat ${MANIFEST_FILE}"

                            // 5. Git Commit & Push
                            sh "git add ${MANIFEST_FILE}"
                            sh "git commit -m 'Update image tag to ${BUILD_NUMBER}'"
                            sh "git push origin main"
                        }
                    }
                }
            }
        }
    }
}