#!groovy

def err_msg = ""
def repo_name = "jenkins_test_laravel"
def git_url = "git@sample.github.com:ohwaki/${repo_name}.git"
def dev_branch = "develop"
def release_branch = "master"

def TARGET_GROUP_ARN = ""
def TARGET_INSTANCE_ID = ""
def TARGET_INSTANCE_PUB_IP = ""
def TARGET_INSTANCE_1_ID = ""
def TARGET_INSTANCE_1_PUB_IP = ""

def parseJson(text) {
    return new groovy.json.JsonSlurperClassic().parseText(text)
}

node("master") {
    // ソースの取得
    // stage("Get Git Resource") {
    //     // カレントディレクトにgitリポジトリが存在するか否かの確認
    //     if(fileExists("./${repo_name}") && fileExists("./${repo_name}/.git")) {
    //         // フェッチ
    //         def FETCH_RESULT = sh(script: "cd ./${repo_name} && git fetch --all", returnStatus: true) == 0
    //         if( ! FETCH_RESULT) {
    //             // throw error
    //             error "fetchに失敗しました"
    //         }
    //         // gitがある場合はpull
    //         def PULL_RESULT = sh(script: "cd ./${repo_name} && git pull --all", returnStatus: true) == 0
    //         if( ! PULL_RESULT) {
    //             error "pullに失敗しました"
    //         }
    //         // ブランチの切替
    //         def CHECKOUT_RESULT = sh(script: "cd ./${repo_name} && git checkout ${release_branch}", returnStatus: true) == 0
    //         if( ! CHECKOUT_RESULT) {
    //             // throw error
    //             error "checkoutに失敗しました"
    //         }
    //     } else {
    //         // gitがない場合はclone
    //         def CLONE_RESULT = sh(script: "git clone ${git_url} ${repo_name}", returnStatus: true) == 0
    //         if( ! CLONE_RESULT) {
    //             error "cloneに失敗しました"
    //         }
    //     }
    // }

    // 2号機を作成
    stage("Get AWS Instance") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 ec2 describe-instances \
            --filter "Name=tag:Name, Values=test_jenkins_2"
        /$
        def AWS_RESULT = sh (script: command, returnStdout: true)
        print AWS_RESULT
        def RESULT_ARRAY = parseJson(AWS_RESULT)
        if(RESULT_ARRAY['Reservations']) {
            TARGET_INSTANCE_ID = RESULT_ARRAY['Reservations']['Instances'][0]['InstanceId'][0]
            TARGET_INSTANCE_PUB_IP = RESULT_ARRAY['Reservations']['Instances'][0]['PublicIpAddress'][0]
            // 起動しているかどうかチェック
            if(RESULT_ARRAY['Reservations']['Instances'][0]['State']['Code'][0] == 16) {
                return
            }
            // 起動していない場合インスタンスの名前を変更する
            def num = (int)(Math.random()*1000)
            command = $/
                /usr/bin/aws --region ap-northeast-1 ec2 create-tags --resources ${TARGET_INSTANCE_ID} --tags '[{"Key": "Name", "Value": "old_test_jenkins${num}"}]'
            /$
            def SET_INSTANCE_NAME_RESULT = sh (script: command, returnStdout: true)
        }

        // 新規にイメージからインスタンスを作成
        command = $/
            /usr/bin/aws --region ap-northeast-1 ec2 run-instances \
            --image-id ami-080f81166e71bc93b --key-name private_ohwaki --count 1 --security-group-ids sg-0b67c1ca57618a603  --instance-type t2.micro
        /$
        def CREATE_INSTANCE_RESULT = sh (script: command, returnStdout: true)
        print CREATE_INSTANCE_RESULT
        if( ! CREATE_INSTANCE_RESULT) {
            // throw error
            error "instanceの作成に失敗しました"
        }
        RESULT_ARRAY = parseJson(CREATE_INSTANCE_RESULT)
        TARGET_INSTANCE_ID = RESULT_ARRAY['Instances'][0]['InstanceId']
        TARGET_INSTANCE_PUB_IP = RESULT_ARRAY['Instances'][0]['PublicIpAddress']
        print TARGET_INSTANCE_ID
        // タグ名を作成
        command = $/
            /usr/bin/aws --region ap-northeast-1 ec2 create-tags --resources ${TARGET_INSTANCE_ID} --tags '[{"Key": "Name", "Value": "test_jenkins_2"}]'
        /$
        def CREATE_INSTANCE_TAG_RESULT = sh (script: command, returnStdout: true)
        error "instanceを作成中です。runningになってから再度実行してください。"
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

    // 1号機を取得
    stage("Get AWS Instance_1") {
        def command_1 = $/
            /usr/bin/aws --region ap-northeast-1 ec2 describe-instances \
            --filter "Name=tag:Name, Values=test_jenkins_1"
        /$
        def AWS_RESULT = sh (script: command_1, returnStdout: true)
        print AWS_RESULT
        def RESULT_ARRAY = parseJson(AWS_RESULT)
        TARGET_INSTANCE_1_ID = RESULT_ARRAY['Reservations']['Instances'][0]['InstanceId'][0]
        TARGET_INSTANCE_1_PUB_IP = RESULT_ARRAY['Reservations']['Instances'][0]['PublicIpAddress'][0]
    }

    stage('AnsibleTest') {
        // ターゲットのIPを変更
        sh "sed -ri 's/target_host/${TARGET_INSTANCE_PUB_IP}/g' /var/lib/jenkins/workspace/jenkins_test_laravel@script/ansible/hosts"
        sh "sed -ri 's/target_1_host/${TARGET_INSTANCE_1_PUB_IP}/g' /var/lib/jenkins/workspace/jenkins_test_laravel@script/ansible/hosts"
        // サーバを初期設定
        sh "cd /var/lib/jenkins/workspace/jenkins_test_laravel@script/ansible && ansible-playbook -i hosts Ansiblefile.yml -u ec2-user --private-key='~jenkins/.ssh/private_ohwaki.pem'"
    }

    stage("Set AWS Instance To Tatget") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 register-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=${TARGET_INSTANCE_ID}
        /$
        def SET_INSTANCE_RESULT = sh(script: command, returnStatus: true) == 0
        if( ! SET_INSTANCE_RESULT) {
            // throw error
            error "ec2のalb接続に失敗しました"
        }
    }

    // 1号機を切り離し
    stage("Remove AWS Instance To Tatget") {
        def command_1 = $/
            /usr/bin/aws --region ap-northeast-1 ec2 describe-instances \
            --filter "Name=tag:Name, Values=test_jenkins_1"
        /$
        def AWS_RESULT = sh (script: command_1, returnStdout: true)
        print AWS_RESULT
        def RESULT_ARRAY = parseJson(AWS_RESULT)
        TARGET_INSTANCE_1_ID = RESULT_ARRAY['Reservations']['Instances'][0]['InstanceId'][0]
        TARGET_INSTANCE_1_PUB_IP = RESULT_ARRAY['Reservations']['Instances'][0]['PublicIpAddress'][0]

        def command_2 = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 deregister-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=${TARGET_INSTANCE_1_ID}
        /$
        def REMOVE_INSTANCE_RESULT = sh(script: command_2, returnStatus: true) == 0
        print REMOVE_INSTANCE_RESULT
        if( ! REMOVE_INSTANCE_RESULT) {
            // throw error
             error "ec2のalb切り離しに失敗しました"
        }
    }

    // 1号機を更新
    stage("Ssh") {
        sh "cd /var/lib/jenkins/workspace/jenkins_test_laravel@script/ansible && ansible-playbook -i hosts server_restart.yml -u ec2-user --private-key='~jenkins/.ssh/private_ohwaki.pem'"
    }

    // 1号機をターゲットに追加
    stage("Set AWS Instance To Tatget") {
        def command = $/
            /usr/bin/aws --region ap-northeast-1 elbv2 register-targets \
            --target-group-arn ${TARGET_GROUP_ARN} --targets Id=${TARGET_INSTANCE_1_ID}
        /$
        def SET_INSTANCE_RESULT = sh(script: command, returnStatus: true) == 0
        if( ! SET_INSTANCE_RESULT) {
            // throw error
            error "ec2のalb接続に失敗しました"
        }
    }
}