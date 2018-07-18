## 開発環境構築

```
 sudo yum -y install git
 sudo yum install java-1.8.0-openjdk
 sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
 sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
 sudo yum install jenkins
 sudo cat << EOS > /etc/sysconfig/jenkins
 JENKINS_PORT="8082"
 EOS
 sudo service jenkins start
 cd /var/lib/jenkins/
 mkdir .ssh
 sudo chown -R jenkins:jenkins .ssh
 ssh-keygen -t rsa // ここで生成した公開鍵を github に登録する
```