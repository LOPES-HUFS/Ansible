# 라즈베리파이을 이용한 앤서블(Ansible) 고급 기초

이제 마지막 기초입니다. 앞선 글은 다음과 같습니다.

- [라즈베리파이를 이용한 Ansible 기초](https://github.com/LOPES-HUFS/Ansible/blob/main/Raspberry_Pi_intro/doc_1.md)
- [라즈베리파이를 이용한 앤서블의 본격적인 기초](https://github.com/LOPES-HUFS/Ansible/blob/main/Raspberry_Pi_intro/doc_2.md)

앞 글, [라즈베리파이를 이용한 앤서블의 본격적인 기초](https://github.com/LOPES-HUFS/Ansible/blob/main/Raspberry_Pi_intro/doc_2.md)에서 특정 부분(패스워드)만 암화화해서 앤서블의 `hello world!`인 `ping`을 해봤습니다.

```bash
❯ ansible all -m ping --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 
lopes.hufs.ac.kr | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

그리고 `df -h`이라는 명령어도 암호화된 파일을 이용해서 실행해봤습니다.

```bash
#PW: 1234
❯ ansible all -a 'df -h' --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 
raspberrypi.local | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       117G  1.3G  111G   2% /
devtmpfs        430M     0  430M   0% /dev
tmpfs           463M     0  463M   0% /dev/shm
tmpfs           463M   12M  451M   3% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           463M     0  463M   0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   49M  204M  20% /boot
tmpfs            93M     0   93M   0% /run/user/1000
```

이번에는 `ansible-playbook`을 이용하여 우리가 따로 따로 실행했던, `ping`과 `df -h`를 한꺼번에 처리해보겠습니다. `ping_and_df.yml`라는 이름을 가진 파일을 만들어서, 다음과 같이 입력합니다.

```yaml
- hosts: all
  user: pi
  tasks:
    - name: Ping all hosts
      ping:
    - name: Let's see how much disk is left.
      shell: df -h
      register: df_meg
    - debug: var=df_meg["stdout_lines"]
```

이것을 실행하려면 다음과 같이 하시면 됩니다.

```bash
ansible-playbook ping_and_df.yml -i new_hosts
```

실행화면은 다음과 같습니다.

```bash
❯ ansible-playbook ping_and_df.yml -i new_hosts

PLAY [all] ****************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [raspberrypi.local]

TASK [Ping all hosts] *****************************************************************************************************************************************
ok: [raspberrypi.local]

TASK [Let's see how much disk is left.] ***********************************************************************************************************************
changed: [raspberrypi.local]

TASK [debug] **************************************************************************************************************************************************
ok: [raspberrypi.local] => {
    "login[\"stdout_lines\"]": [
        "Filesystem      Size  Used Avail Use% Mounted on",
        "/dev/root        59G  1.3G   55G   3% /",
        "devtmpfs        430M     0  430M   0% /dev",
        "tmpfs           463M     0  463M   0% /dev/shm",
        "tmpfs           463M  6.5M  456M   2% /run",
        "tmpfs           5.0M  4.0K  5.0M   1% /run/lock",
        "tmpfs           463M     0  463M   0% /sys/fs/cgroup",
        "/dev/mmcblk0p1  253M   47M  206M  19% /boot",
        "tmpfs            93M     0   93M   0% /run/user/1000"
    ]
}

PLAY RECAP ****************************************************************************************************************************************************
raspberrypi.local          : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

물론 암호화된 인벤토리 파일인 `encrypted_variable_hosts.yaml`을 가지고도 실행 할 수 있습니다.

```bash
ansible-playbook ping_and_df.yml --ask-vault-password -i encrypted_variable_hosts.yaml
```

실행 과정은 다음과 같습니다.

```bash
#PW: 1234
❯ ansible-playbook ping_and_df.yml --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 

PLAY [all] ***********************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [raspberrypi.local]

TASK [Ping all hosts] ************************************************************************************************************************
ok: [raspberrypi.local]

TASK [Let's see how much disk is left.] ******************************************************************************************************
changed: [raspberrypi.local]

TASK [debug] *********************************************************************************************************************************
ok: [raspberrypi.local] => {
    "login[\"stdout_lines\"]": [
        "Filesystem      Size  Used Avail Use% Mounted on",
        "/dev/root       117G  1.3G  111G   2% /",
        "devtmpfs        430M     0  430M   0% /dev",
        "tmpfs           463M     0  463M   0% /dev/shm",
        "tmpfs           463M  6.2M  456M   2% /run",
        "tmpfs           5.0M  4.0K  5.0M   1% /run/lock",
        "tmpfs           463M     0  463M   0% /sys/fs/cgroup",
        "/dev/mmcblk0p1  253M   49M  204M  20% /boot",
        "tmpfs            93M     0   93M   0% /run/user/1000"
    ]
}

PLAY RECAP ***********************************************************************************************************************************
raspberrypi.local          : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

`ansible-playbook`을 이용하면 위와 같이 한꺼번에 두 가지 과제를 처리할 수 있게 됩니다. 우리가 서버에 들어가서 일일히 변경하는 것이 아니라 서버에서 할 과제를 쭉 적은 다음 그것을 순서대로 각본대로 쭉 실행되는 것이죠. 아주 단순하게 설명하자면, `ping_and_df.yml`이라는 각본에, `tasks`에서 첫번째 줄 `name`은 과제 이름을 말하는 것이고 두 번째 줄인 `shell`과 `ping`은 그 과제에서 할 것을 쓴 것입니다. 물론 정확하게 설명하자면 더 복잡하게 설명해야 합니다.

## 앤서블(Ansible)을 이용하여 라즈베리파이에 nginx 설치하기

이제 마지막으로 실전에서 사용할 만한 것을 해보겠습니다. 모든 패키지를 업데이트하고 업그래이드한 다음, nginx(이하 엔진엑스) 웹서버를 설치하고, 이를 작동해 보겠습니다. `nginx.yml`라는 이름을 가진 파일을 만들어서, 다음과 같이 입력합니다.

```yaml
---
- hosts: all
  user: pi
  tasks:
    - name: Update and upgrade apt packages
      become: true
      apt:
        upgrade: yes
        update_cache: yes
        cache_valid_time: 86400 #One day
    - name: ensure nginx is at the latest version
      become: true
      apt: name=nginx state=latest
    - name: start nginx
      service:
          name: nginx
          state: started

```

이렇게 입력한 것은 다음과 같이 실행할 수 있습니다.

```bash
ansible-playbook nginx.yml --ask-vault-password -i encrypted_variable_hosts.yaml
```

실행 과정은 다음과 같습니다.

```bash
#PW: 1234
❯ ansible-playbook nginx.yml --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 

PLAY [all] ***********************************************************************************************************************************
****************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************
ok: [raspberrypi.local]

TASK [Update and upgrade apt packages] ************************************************************************************************************************
changed: [raspberrypi.local]

TASK [ensure nginx is at the latest version] ******************************************************************************************************************
changed: [raspberrypi.local]

TASK [start nginx] ********************************************************************************************************************************************
ok: [raspberrypi.local]

PLAY RECAP ****************************************************************************************************************************************************
raspberrypi.local          : ok=4    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

라즈베리파이에 설치된 엔진엑스가 잘 작동하는지 살펴보려면 아래와 같이 현재 라즈베리파이 IP를 웹브라우저 주소창에 입력하시면 됩니다. 뭐 `Welcome to nginx!`가 제목으로 나오는 페이지가 나온다면 잘 설치된 것입니다. 참고로 `become: true`은 `sudo`라고 생각하시면 됩니다. 물론 정확한 설명은 아닙니다.

[http://raspberrypi.local](hhttp://raspberrypi.local)

워 이렇게 번잡하게 설치하는지 궁굼하실 수도 있겠지만, 서버 1개에 설치하는 것은 그리 어렵지 않지만, 같은 것을 서버 여러 대에 설치하고자 한다면, 이와 같이 엔서블을 이용하는 것이 훨씬 더 효과적입니다. 그리고 한 번 잘 돌아간 엔서블 코드는 오타도 없는 코드이니, 직접 터미널에서 작업할 때 오타의 위험에서도 벗어나실 수 있습니다.

## 앤서블(Ansible)을 이용하여 라즈베리파이에 파일 다운받기

진짜 마지막으로 현재 관리하고 있는 라즈베리파이 서버로 파일을 복사해 보겠습니다. 하둡 파일을 다운받아 보내겠습니다. 우선 현재 앤서블(Ansible)를 실행하고 있는 컴퓨터에 파일을 다운받겠습니다.

```bash
wget https://downloads.apache.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
```

자 앞에서 받은 파일을 라즈베리파이로 복사해 보겠습니다.

```bash
ansible all -m copy -a "src=hadoop-3.2.2.tar.gz  dest=~/" -i new_hosts
```

다음과 같이 작동됩니다.

```bash
❯ ansible all -m copy -a "src=hadoop-3.2.2.tar.gz  dest=~/" -i new_hosts

raspberrypi.local | CHANGED => {
    "changed": true,
    "checksum": "f4fa4e95192c6bb33258b8bd7adb0267a306ba27",
    "dest": "/home/pi/hadoop-3.2.2.tar.gz",
    "gid": 1000,
    "group": "pi",
    "md5sum": "e7cebb6420657030539dc644ed7ee1f0",
    "mode": "0644",
    "owner": "pi",
    "size": 395448622,
    "src": "/home/pi/.ansible/tmp/ansible-tmp-1616143409.119393-82690-53250984102454/source",
    "state": "file",
    "uid": 1000
}
```

이번에는 각본(playbook)을 만들어서 복사해 보겠습니다. `copy.yml`라는 이름을 가진 파일을 만들어서, 다음과 같이 입력합니다.

```yaml
---
- hosts: all
  remote_user: root
  tasks:
  - name: store file to remote server
    copy:
      src: hadoop-3.2.2.tar.gz 
      dest: ~/hadoop-3.2.2.tar.gz 

```

앞에서 만든 파일을 `ansible-playbook copy.yml -i new_hosts`으로 실행합니다. 결과는 다음과 같습니다.

```bash
❯ ansible-playbook copy.yml -i new_hosts

PLAY [all] ******************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************
ok: [raspberrypi.local]

TASK [store file to remote server] ******************************************************************************************************************************
changed: [raspberrypi.local]

PLAY RECAP ******************************************************************************************************************************************************
raspberrypi.local          : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

엔서블을 이용해서 라즈베리파이 서버에 잘 복사되어 있는지 확인해보니, 잘 작동한 것 같습니다.

```bash
❯ ansible all -a 'ls -l' --ask-vault-password -i encrypted_variable_hosts.yaml 
Vault password: 
raspberrypi.local | CHANGED | rc=0 >>
total 386188
-rw-r--r-- 1 pi pi 395448622 Jul 21 17:13 hadoop-3.2.2.tar.gz
```

앞에서는 현재 앤서블(Ansible)를 실행하고 있는 컴퓨터에 파일을 다운받은 다음 이것을 라즈베리파이 서버로 복사했습니다. 이번에는 직접 라즈베리파이 서버로 다운받아보겠습니다. 그러면서 다운받은 파일이 해킹된 것인지 아닌지 `sha256`파일을 이용하여 체크도 해보겠습니다.

우선 라즈베리파이 서버에서 다운받은 파일을 지웁니다. 아래에서 `[WARNING]`가 뜨는 것은 `rm`을 이용해서 직접 지우는 것보다, `file module`을 사용하는 것이 좋으니 그것을 사용하라고 하는 것입니다. 이건 연습을 위한 것이니 `rm`을 이용해서 직접 지웠습니다.

```bash
❯ ansible all -a 'rm hadoop-3.2.2.tar.gz' --ask-vault-password -i encrypted_variable_hosts.yaml 
Vault password: 
[WARNING]: Consider using the file module with state=absent rather than running 'rm'.  If you need to use command because file is
insufficient you can add 'warn: false' to this command task or set 'command_warnings=False' in ansible.cfg to get rid of this message.
raspberrypi.local | CHANGED | rc=0 >>
```

지워진 것을 다음과 같이 확인할 수 있습니다.

```bash
❯ ansible all -a 'ls -l' --ask-vault-password -i encrypted_variable_hosts.yaml 
Vault password: 
raspberrypi.local | CHANGED | rc=0 >>
total 0
```

`download_file_with_checksum.yaml`라는 이름을 가진 파일을 만들어서, 다음과 같이 입력합니다. 여기서 `url`은 다운받을 파일 주소, `dest`다운받을 장소, `checksum`은 문제가 없는 파일인지 검사할 때 사용할 파일이 있는 주소입니다. 좀 더 자세한 내용은 아래 링크를 참고하세요.

- [ansible.builtin.get_url – Downloads files from HTTP, HTTPS, or FTP to node — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/get_url_module.html)ng

```yaml
---
- hosts: all
  remote_user: root
  tasks:
  - name: Download file with checksum url (sha512)
    get_url:
      url: https://downloads.apache.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz
      dest: ~/hadoop-3.2.2.tar.gz
      checksum: sha512:https://downloads.apache.org/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz.sha512

```

앞에서 만든 파일을 다음 명령어로 실행합니다.

```bash
ansible-playbook download_file_with_checksum.yaml --ask-vault-password -i encrypted_variable_hosts.yaml
```

으로 실행합니다. 결과는 다음과 같습니다.

```bash
❯ ansible-playbook download_file_with_checksum.yaml --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 

PLAY [all] ***********************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [raspberrypi.local]

TASK [Download file with checksum url (sha512)] **********************************************************************************************
changed: [raspberrypi.local]

PLAY RECAP ***********************************************************************************************************************************
raspberrypi.local          : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

라즈베리파이에 파일이 다음과 같이 잘 들어온 것을 확인할 수 있습니다. 물론 해킹당하지 않은 것도 확인한 것이구요.

```bash
❯ ansible all -a 'ls -l' --ask-vault-password -i encrypted_variable_hosts.yaml
Vault password: 
raspberrypi.local | CHANGED | rc=0 >>
total 386188
-rw-r--r-- 1 pi pi 395448622 Jul 22 15:13 hadoop-3.2.2.tar.gz
```
