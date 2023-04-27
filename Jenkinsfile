node {
  stage ('======== Clone repository ========') {
    checkout scm #scm이란 여기서는 git을 말하는 것 환경변수에서 설정한 git repo를 읽어오는 과정
  }
  stage('======== Build image ========') {
    sh "git switch ~~~" #~~~에는 Dockerfile이 있는 branch를 적어준다. 
    sh "git pull" #혹시 그 사이에 git에 업로드 된 사항이 있을 수 있으니 git pull을 해주어 항상 최신화를 먼저 해주어야 한다.
    app = docker.build("test/nginx") #docker plugin을 설치했다면 docker.build 함수를 사용할 수 있다. 이는 docker build -t test/nginx . 명령어를 쳐준다고 보면된다.
  }
  stage('======== Push image ========') {
    docker.withRegistry('https://shryu-ncr.kr.ncr.ntruss.com', 'ncr') { #docker에 로그인 하는 과정 여기서 ncr은 jenkins credential(docker 정보가 담긴 credential)의 ID이다. 만든 걸로 바꿔라.
       app.push("${env.BUILD_NUMBER}") #git에 build 넘버대로 image에 tag를 붙여서 push하는 과정
       app.push("latest")
    }
  }
  stage('======== Update YAML file ========') {
    sh "git switch yaml" #나는 branch 'yaml'에 nginx.yaml파일을 올려두었다. 왜냐면 webhook을 사용할 겅우 이 소스 파일 업로드를 할때만 webhook기능이 동작하면 되는데 yaml파일이 소스파일과 같은 branch를 쓴다면 이 stage가 끝난뒤 webhook기능이 또 동작하여 무한 빌드가 된다. 이 때문에 index.html(소스파일),Dockerfile(소스파일과 같은위치에 있어야됨),Jenkinsfile 을 같은 branch에 넣어주고 yaml파일은 다른 branch에 넣어준다.
    sh "git pull" #최신화
    sh "git config --global user.email 'shryu@cloit.com'" #config는 git과의 연동이 안되어있을 경우나 최초에 해주어야하는 작업이기 때문에 넣어주는게 속 편하다.
    sh "git config --global user.name 'shryu'"
    sh "sed -i s%shryu-ncr.kr.ncr.ntruss.com/test/nginx:.*%shryu-ncr.kr.ncr.ntruss.com/test/nginx:${env.BUILD_NUMBER}%g nginx.yaml" #yaml파일 내에 이미지 tag를 build number로 변경해주는 sed명령어
    sh "cat nginx.yaml | grep image:"
    sh "git add nginx.yaml"
    sh "git commit -m 'image tag update ${env.BUILD_NUMBER}'"
    sh "git push -u origin yaml"
  }
}
