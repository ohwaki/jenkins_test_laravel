#!groovy

def err_msg = ""
def repo_name = "jenkins_test_laravel"
def git_url = "git@sample.github.com:ohwaki/${repo_name}.git"
def dev_branch = "develop"
def release_branch = "master"

def TARGET_GROUP_ARN = ""

def parseJson(text) {
    return new groovy.json.JsonSlurperClassic().parseText(text)
    // return new JsonSlurper().parse(text)
}

node {
    // ソースの取得
    stage("Get Git Resource") {
        // カレントディレクトにgitリポジトリが存在するか否かの確認
        if(fileExists("./${repo_name}") && fileExists("./${repo_name}/.git")) {
            // フェッチ
            def FETCH_RESULT = sh(script: "cd ./${repo_name} && git fetch --all", returnStatus: true) == 0
            if( ! FETCH_RESULT) {
                // throw error
                error "fetchに失敗しました"
            }
            // gitがある場合はpull
            def PULL_RESULT = sh(script: "cd ./${repo_name} && git pull --all", returnStatus: true) == 0
            if( ! PULL_RESULT) {
                error "pullに失敗しました"
            }
            // ブランチの切替
            def CHECKOUT_RESULT = sh(script: "cd ./${repo_name} && git checkout ${release_branch}", returnStatus: true) == 0
            if( ! CHECKOUT_RESULT) {
                // throw error
                error "checkoutに失敗しました"
            }
        } else {
            // gitがない場合はclone
            def CLONE_RESULT = sh(script: "git clone ${git_url} ${repo_name}", returnStatus: true) == 0
            if( ! CLONE_RESULT) {
                error "cloneに失敗しました"
            }
        }
    }

    stage("Get AWS Instance") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 ec2 describe-instances
        /$
        def AWS_RESULT = sh (script: command, returnStdout: true)
        print AWS_RESULT
        if( ! AWS_RESULT) {
            // throw error
            error "instance情報取得に失敗しました"
        }
    }

    stage("Create AWS ALB") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 create-load-balancer \
            --name my-load-balancer --subnets subnet-01dfcb48 subnet-079fd95c
        /$
        def CREATE_ALB_RESULT = sh(script: command, returnStdout: true)
        print CREATE_ALB_RESULT
        if( ! CREATE_ALB_RESULT) {
            // throw error
            error "albの作成に失敗しました"
        }
    }

    stage("Create AWS Tatget") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 create-target-group \
            --name my-targets --protocol HTTP --port 80 --vpc-id vpc-5398d634
        /$
        def CREATE_TARGET_RESULT = sh(script: command, returnStdout: true)
        print CREATE_TARGET_RESULT
        if( ! CREATE_TARGET_RESULT) {
            // throw error
            error "targetの作成に失敗しました"
        }
        def RESULT_ARRAY = parseJson(CREATE_TARGET_RESULT)
        TARGET_GROUP_ARN = RESULT_ARRAY['TargetGroups'][0]['TargetGroupArn']
    }

    stage("Set AWS Instance To Tatget") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 register-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=i-0519d9b600e7ac00d
        /$
        def SET_INSTANCE_RESULT = sh(script: command, returnStatus: true) == 0
        print SET_INSTANCE_RESULT
        if( ! SET_INSTANCE_RESULT) {
            // throw error
            error "ec2のalb接続に失敗しました"
        }
    }

    stage("Remove AWS Instance To Tatget") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 deregister-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=i-0519d9b600e7ac00d
        /$
        def REMOVE_INSTANCE_RESULT = sh(script: command, returnStatus: true) == 0
        print REMOVE_INSTANCE_RESULT
        if( ! REMOVE_INSTANCE_RESULT) {
            // throw error
             error "ec2のalb切り離しに失敗しました"
        }
    }

    stage("Ssh") {
        sshagent([]) {
            sh 'ssh -t -t test_ec2 pwd'
            sh 'ssh -t -t test_ec2 sudo service httpd status'
            sh 'ssh -t -t test_ec2 sudo service httpd restart'
        }
    }

    stage("Set AWS Instance") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 register-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=i-0519d9b600e7ac00d
        /$
        def SET_INSTANCE_RESULT = sh(script: command, returnStatus: true) == 0
        if( ! SET_INSTANCE_RESULT) {
            // throw error
            error "ec2のalb接続に失敗しました"
        }
    }
}