# ansible_playground_20221106

構成管理ツール Ansible のおためし

## 実行環境

- Ubuntu 20.04 (Host OS)
- VirtualBox 7.0
  - <https://www.virtualbox.org/wiki/Linux_Downloads>
  - ハードウェア仮想化機能 Intel VT-x または AMD-V が有効化された環境が必要
    - KVM等による仮想環境として提供される通常のVPS等の環境では動作不可
- Vagrant 2.3
  - <https://developer.hashicorp.com/vagrant/downloads>
- Ansible 2.12
  - <https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html#installing-ansible-on-ubuntu>

## 何をするか

VagrantでVM（Ubuntu 20.04 with sshd）を立て、
AnsibleでDocker/Docker Composeがインストールされた環境を宣言し、適用する。

## 参考

- <https://docs.ansible.com/ansible/latest/getting_started/get_started_inventory.html>
- <https://docs.ansible.com/ansible/latest/getting_started/get_started_playbook.html>
- <https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html>
- <https://blog.apar.jp/linux/5154/>
- <https://docs.ansible.com/ansible/latest/user_guide/become.html>
- <https://www.digitalocean.com/community/tutorials/how-to-use-ansible-to-install-and-set-up-docker-on-ubuntu-20-04>
- <https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04>
- <https://docs.docker.com/engine/install/ubuntu/>
- <https://www.shellhacks.com/ansible-sudo-a-password-is-required/>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_key_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_repository_module.html>
- <https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html>

## 実行例

```shell
# Remove the VM's SSH server key fingerprint from the host's known_hosts (required for re-created VMs)
ssh-keygen -f "$HOME/.ssh/known_hosts" -R "192.168.56.10"

# Generate a ssh key pair (without password) to sign in the Vagrant VM
mkdir -p secrets
ssh-keygen -f secrets/key -P ""

# Initialize the Vagrant VM
vagrant up

# Execute the playbook
ansible-playbook -i inventory.yaml playbook.yaml

# Open a ssh connection into the VM
vagrant ssh

# Check the Docker environment inside the VM installed by Ansible
sudo docker version
sudo docker compose version

# Remove the VM for cleaning up
vagrant destroy
```

## 問題点

- デフォルトでbecome環境なので、各Taskでbecomeが使えない（はず）
  - become in becomeができず、ログインユーザでbecomeを試みる挙動になる（はずの）ため
  - 各Taskでbecomeを指定しても、`vagrant -> root -> anotheruser`ではなく、`vagrant -> anotheruser`となって失敗するはず
    - becomeがどのように実行されるかによる？ `sudo -l anotheruser`ならば成功しそう
  - `ansible_become`または`become`を、`inventory.yaml`または`playbook.yaml`のどちらに記述するべきか
    - Playbookを書くにあたって、`become`が使用できない、という制約を無視できない
    - `inventory.yaml`は、サーバインスタンス固有の性質に依存する設定を記述し、`playbook.yaml`はインスタンスにできるだけ依存しない形で記述する（OSとかは無理だが）のがよさそうだと認識している（あるいは、Playbookの実行に必要な接続に関わる設定が`inventory.yaml`なのかもしれない）
      - しかし、`become`が必要とされるのは、サーバインスタンス固有の性質なので`inventory.yaml`に記述しているが、Playbookは`become`が設定されているという前提で記述することになる、という実態の乖離がある
- ログインユーザ`vagrant`がパスワードレスsudo権限を持っている（実質`PermitRootLogin yes`と同等では？）
- Vagrant Ubuntu Boxは、デフォルトで`ubuntu`というパスワードレスsudoユーザが存在するが、別途`vagrant`ユーザでパスワードレスsudoをセットアップしていて、冗長
  - Vagrantにおけるプロビジョニングは、Ansibleの使用という点において本質的ではないと思われるので、`ubuntu`ユーザを使うように変更してもいいかもしれない
