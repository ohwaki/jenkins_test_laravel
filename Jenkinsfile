#!groovy

def err_msg = ""
def repo_name = "jenkins_test_laravel"
def git_url = "git@sample.github.com:ohwaki/${repo_name}.git"
def dev_branch = "dev"
def release_branch = "master"

node {
    // ソースの取得
    stage("get resource") {
        // カレントディレクトにgitリポジトリが存在するか否かの確認
        if(fileExists("./${repo_name}") && fileExists("./${repo_name}/.git")) {
            // フェッチ
            def FETCH_RESULT = sh(script: "cd ./${repo_name} && git fetch --all", returnStatus: true) == 0
            if(!FETCH_RESULT) {
                // throw error
                error "fetchに失敗しました"
            }
            // gitがある場合はpull
            def PULL_RESULT = sh(script: "cd ./${repo_name} && git pull --all", returnStatus: true) == 0
            if(!PULL_RESULT) {
                error "pullに失敗しました"
            }
            // ブランチの切替
            def CHECKOUT_RESULT = sh(script: "cd ./${repo_name} && git checkout ${dev_branch}", returnStatus: true) == 0
            if(!CHECKOUT_RESULT) {
                // throw error
                error "checkoutに失敗しました"
            }
        } else {
            // gitがない場合はclone
            def CLONE_RESULT = sh(script: "git clone ${git_url} ${repo_name}", returnStatus: true) == 0
            if(!CLONE_RESULT) {
                error "cloneに失敗しました"
            }
        }

        // phpライブラリのインストール
        stage("install libs") {
            withEnv(["PATH+NODE=${JENKINS_HOME}/.nvm/versions/node/v6.9.5/bin/"]) {
                def NPM_RESULT = sh(script: "cd ./${repo_name} && npm install", returnStatus: true) == 0
                if(!NPM_RESULT) {
                    error "npm installに失敗しました"
                }
            }
        }
    }
}