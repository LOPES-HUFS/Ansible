# 라즈베리파이을 이용한 Ansible 기초

이 글은 라즈베리파이를 가지고 Ansible을 공부하면서 작업하는 코드를 모두 넣은 것입니다.

라즈베리파이를 설치하는 가장 쉬운 방법은 **Raspberry Pi Imager**을 이용하는 것입니다. 설치하는 방법과 사용 방법을 잘 모르신다면, 다음 링크를 참고하세요.

[How to use Raspberry Pi Imager | Install Raspberry Pi OS to your Raspberry Pi (Raspbian) - YouTube](https://www.youtube.com/watch?v=ntaXWS8Lk34)

앞의 동영상에도 언급되고 있지만, Raspberry Pi Imager v1.6부터는 ‘Ctrl-Shift-X’이라는 단축키를 가지고 advanced options을 설정할 수 있습니다. 이 옵션(options)에는 다음과 같은 것이 들어 있습니다. 이 옵션에 들어 있는 것들이 라즈베리파이의 설정을 쉽게 할 수 있도록 만들어 줍니다.

- Set hostname
- Enable SSH
- Configure wifi
- Set locale settings

여기서 라즈베리파이를 유선랜으로 연결하여 서버로 사용할 때 필요한 옵션은 '호스트 이름 설정(Set hostname)', 'SSH 활성화(Enable SSH)'입니다. 원래는 이미지 파일을 다운 받아서 메모리 카드에 넣은 다음, 이를 라즈베리파이에 설치한 다음, 호스트 이름 설정을 설정하고 SSH 활성화를 해야했는데, 이제는 Raspberry Pi Imager에서 바로 설정한 다음, 이미지 파일을 다운 받아서 메모리 카드에 넣은 후 이를 라즈베리파이에 설치하면 설정이 바로 적용됩니다.

여기서는 호스트 이름은 `raspberrypi.local`으로 설정하고 SSH 활성화를 체크한 다음 비밀번호(password)를 `raspberry`을 설정하였다고 가정하고 글을 쓰겠습니다. 현재 터미널에서 `ping`을 한다면, 다음과 같이 응답할 것입니다.

```bash
❯ ping raspberrypi.local
PING raspberrypi.local (192.168.0.16): 56 data bytes
64 bytes from 192.168.0.16: icmp_seq=0 ttl=64 time=0.504 ms
64 bytes from 192.168.0.16: icmp_seq=1 ttl=64 time=0.742 ms
```

라즈베리파이의 기본 설정은 어느 정도 끝났기 때문에, 이제 Ansible을 사용하기 위한 준비 작업을 하겠습니다.우선 `sshpass`을 설치해야 합니다. `ssh`을 사용하지 않고 `sshpass` 사용하는 이유는 다음과 같습니다.

- [SSH password automation in Linux with sshpass | Enable Sysadmin](https://www.redhat.com/sysadmin/ssh-automation-sshpass)
- [[Ansible]Centos8 에서 sshpass 설치하기](https://songsiaix.tistory.com/m/40)

`sshpass`은 래드햇에서는 `yum install sshpass`, `sudo apt install sshpass` 맥에서는 아래 링크를 참고합니다.

- [맥에서 sshpass 설치하기 | Epistemology](http://www.epistemology.pe.kr/2021/03/09/1361)

`sshpass`로 사용하시려면, `public SSH key`를 사용해야 합니다. 아래 링크를 참고하시면 됩니다.

- [Ansible Setting Up SSH (Raspberry Pi) – Geek Tech Stuff](https://geektechstuff.com/2019/06/27/ansible-setting-up-ssh-raspberry-pi/)

`sshpass`을 연습해보겠습니다. 참고로 `sh-keygen -R raspberrypi.local`은 `raspberrypi.local`을 사용했던 기록을 지우는 명령어입니다. 만약 굳이 할 필요는 없습니다.

```bash
# ssh pi@raspberrypi.local
# PW: raspberry
ssh-keygen -R raspberrypi.local
#sshpass -p runit ssh root@pr -o StrictHostKeyChecking=no
sshpass -p raspberry ssh pi@raspberrypi.local -o StrictHostKeyChecking=no 'df -h'
```

실행 결과는 아래와 같습니다.

```bash
Warning: Permanently added 'raspberrypi.local,fe80::3f18:10dc:e523:994c%bridge100' (ECDSA) to the list of known hosts.
Filesystem      Size  Used Avail Use% Mounted on
/dev/root        59G  1.3G   55G   3% /
devtmpfs        430M     0  430M   0% /dev
tmpfs           463M     0  463M   0% /dev/shm
tmpfs           463M   12M  451M   3% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           463M     0  463M   0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   47M  206M  19% /boot
tmpfs            93M     0   93M   0% /run/user/1000
```

자 이제 앤서블의 `Hello World!`인 앤서블 `ping`을 해봅시다! 참고로 라즈베리파이는 `--user=pi`라고 지정해주지 않으면 `root`를 찾아서 올바른 PW를 넣어도 에러가 발생합니다. 그리고 `--inventory 'raspberrypi.local,'`을 설정하는 이유는 현재 따로 `inventory`를 안 만들었기 때문에 문자열로 만들어 준 것입니다. `'raspberrypi.local'` 이렇게 쉼표 없이 넣으면, 에러가 발생하니 주의합니다.

```bash
ansible all -m ping  --ask-pass --user=pi --inventory 'raspberrypi.local,'
```

결과가 다음과 같습니다.

```bash
SSH password: 
[DEPRECATION WARNING]: Distribution debian 10.7 on host raspberrypi.local should use /usr/bin/python3, but is using /usr/bin/python for backward compatibility
 with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in version 2.12. 
Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
raspberrypi.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}

```

`pong`이 나왔기 때문에 잘 된 것입니다. 앞에서 나오는 복잡한 이야기는 파이썬 설정 문제입니다. 다음 링크를 참고합니다. 두 번째 링크를 앞 [DEPRECATION WARNING]에 나온 링크입니다. 해결책은 설정파일에 `ansible_python_interpreter=/usr/bin/python3`을 넣어주면 되는 것 같습니다.

- [[Ansible] host x.x.x.x should use /usr/bin/python3 - 호롤리한 하루](https://gruuuuu.github.io/error/ansible-py-err/#)
- [Interpreter Discovery — Ansible Documentation](https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html)

그냥 다음과 같이 `-e 'ansible_python_interpreter=/usr/bin/python3'`을 명령에 추가해 실행하면, 앞의 [DEPRECATION WARNING]는 나오지 않습니다.

```bash
❯ ansible all -m ping  --ask-pass --user=pi --inventory 'raspberrypi.local,' -e 'ansible_python_interpreter=/usr/bin/python3'
SSH password: 
raspberrypi.local | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

참고로 `ansible-playbook`을 사용할 때는 아래와 같이 하면, 앞의 [DEPRECATION WARNING]가 안 나옵니다.

```bash
ansible-playbook sample-playbook.yml -e 'ansible_python_interpreter=/usr/bin/python3'
```

아래 링크를 참고합니다.

- [Python 3 Support — Ansible Documentation](https://docs.ansible.com/ansible/latest/reference_appendices/python_3_support.html)
