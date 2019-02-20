pipeline {
    agent {
        // 此处设定构建环境，目前可选有
        // java-8,python-2.7,python-3.5,build-essential,ruby-2.3,go-1.7
        label "java-8"
    }
    stages  {
        
        stage("检出") {
            steps {
                sh 'ci-init'
                checkout(
                  [$class: 'GitSCM', branches: [[name: env.GIT_BUILD_REF]], 
                  userRemoteConfigs: [[url: env.GIT_REPO_URL]]]
                )
            }
        }
        // 请在此处补充您的构建过程
    }
}