# gitlab cicd

wget https://mirrors.tuna.tsinghua.edu.cn/gitlab\-ce/yum/el7/gitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm

rpm \-Uvh gitlab\-ce\-14.1.1\-ce.0.el7.x86\_64.rpm

```

GitLab was unable to detect a valid hostname for your instance.
Please configure a URL for your GitLab instance by setting `external_url`
configuration in /etc/gitlab/gitlab.rb file.
Then, you can start your GitLab instance by running the following command:
  sudo gitlab-ctl reconfigure

```

external\_url 'https://192.168.31.120:5566'

gitlab\-ctl reconfigure

```
Warnings:
LD_LIBRARY_PATH was found in the env variables, this may cause issues with linking against the included libraries.

Notes:
Default admin account has been configured with following details:
Username: root
Password: You didn't opt-in to print initial root password to STDOUT.
Password stored to /etc/gitlab/initial_root_password. This file will be cleaned up in first reconfigure run after 24 hours.

NOTE: Because these credentials might be present in your log files in plain text, it is highly recommended to reset the password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.
```

gitlab\-ctl restart

登陆，添加用户

创建项目

增加.readme .cicd.yaml

//方法2\-官网

```
sudo yum install -y curl policycoreutils-python openssh-server perl

sudo systemctl enable sshd
sudo systemctl start sshd

sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo systemctl reload firewalld

```

```
sudo yum install postfix
sudo systemctl enable postfix
sudo systemctl start postfix
```

错误：

> Job for postfix.service failed because the control process exited with error code. See "systemctl status postfix.service" and "journalctl \-xe" for details.

vim /etc/postfix/main.conf

> curl https://packages.gitlab.com/install/repositories/gitlab/gitlab\-ee/script.rpm.sh | sudo bash

sudo EXTERNAL\_URL="http://gitlab.xlsyfx.cn:6677" yum install \-y gitlab\-ee
