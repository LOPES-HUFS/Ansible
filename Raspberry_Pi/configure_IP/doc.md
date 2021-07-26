# 라즈베리파이의 IP를 변경하기

현재 라즈베리파이를 설치하는 가장 쉬운 방법은 **Raspberry Pi Imager**을 이용하는 것입니다. 설치하는 방법과 사용 방법을 잘 모르신다면, 다음 링크를 참고하세요.

[How to use Raspberry Pi Imager | Install Raspberry Pi OS to your Raspberry Pi (Raspbian) - YouTube](https://www.youtube.com/watch?v=ntaXWS8Lk34)

앞의 동영상에도 언급되고 있지만, Raspberry Pi Imager v1.6부터는 ‘Ctrl-Shift-X’이라는 단축키를 가지고 advanced options을 설정할 수 있습니다. 이 옵션(options)에는 다음과 같은 것이 들어 있습니다. 이 옵션에 들어 있는 것들이 라즈베리파이의 설정을 쉽게 할 수 있도록 만들어 줍니다.

- Set hostname
- Enable SSH
- Configure wifi
- Set locale settings

여기서 라즈베리파이를 유선랜으로 연결하여 서버로 사용할 때 필요한 옵션은 '호스트 이름 설정(Set hostname)', 'SSH 활성화(Enable SSH)'입니다. 원래는 이미지 파일을 다운 받아서 메모리 카드에 넣은 다음, 이를 라즈베리파이에 설치한 다음, 호스트 이름 설정을 설정하고 SSH 활성화를 해야했는데, 이제는 Raspberry Pi Imager에서 바로 설정한 다음, 이미지 파일을 다운 받아서 메모리 카드에 넣은 후 이를 라즈베리파이에 설치하면 설정이 바로 적용됩니다.

여기서는 호스트 이름은 `raspberrypi.local`으로 설정하고 SSH 활성화를 체크한 다음 비밀번호(password)를 `raspberry`을 설정하였다고 가정하고 글을 쓰겠습니다.

다음과 같이 하면 

```bash
ansible all -m ping  --ask-pass --user=pi --inventory 'one.local,two.local,three.local,'
```

기본적으로 라즈베리파이를 설치하면, 기본적으로 DHCP 즉[동적 호스트 구성 프로토콜](https://ko.wikipedia.org/wiki/동적_호스트_구성_프로토콜)로 네트웍을 구성합니다. 그러나 라즈베리파이로 서버를 구성하려면 DHCP으로 구성된 동적(dynamic) IP를 사용하는 것보다. 고정된 정적(static) IP를 하는 것이 중요합니다. 라즈베리파이는 `/etc/dhcpcd.conf`을 이용하여 Static IP address를 구성할 수 있습니다. 자세한 내용은 아래 링크를 참고하세요.

- [TCP/IP networking - Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/configuration/tcpip/)


## 라즈베리 서버 3개를 동시에 바꾸기

자, 이제 라즈베리 서버 3개에 os를 잘 설치했으면, 연결이 잘 되는지 테스트 해봅시다.

```bash
ansible all -m ping  --ask-pass --user=pi --inventory 'one.local,two.local,three.local,'
```

결과가 다음과 같이 나온다면, 잘 설치한 것입니다. `SSH password`는 모두 동일하게 `raspberry`을 사용했으니 그걸 입력하시면 됩니다.

```bash
SSH password: 
[DEPRECATION WARNING]: Distribution debian 10.9 on host three.local should use /usr/bin/python3, but is using /usr/bin/python for backward 
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in 
version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
three.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
[DEPRECATION WARNING]: Distribution debian 10.9 on host two.local should use /usr/bin/python3, but is using /usr/bin/python for backward 
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in 
version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
two.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
[DEPRECATION WARNING]: Distribution debian 10.9 on host one.local should use /usr/bin/python3, but is using /usr/bin/python for backward 
compatibility with prior Ansible releases. A future Ansible release will default to using the discovered platform python for this host. See 
https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html for more information. This feature will be removed in 
version 2.12. Deprecation warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.
one.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

이제 호스트 파일을 `hosts.yaml`이라고 만들고 다음 내용을 입력한 다음 ansible을 이용해 봅시다.

```yaml
all:
  hosts:
    one:
      ansible_host: one.local
    two:
      ansible_host: two.local
    three:
      ansible_host: three.local
  vars:
    ansible_python_interpreter: /usr/bin/python3
    ansible_user: pi
    ansible_ssh_pass: raspberry
```

앞의 파일을 만들었으면, 당연히 ansible `hello world!`인 `ping`을 해봅시다. 앞의 파일이 있는 곳에서 다음 명령어를 수행해야 합니다.

```bash
ansible all -m ping -i hosts.yaml
```

아주 간단하게 결과가 다음과 같이 출력됩니다.

```bash
❯ ansible all -m ping -i hosts.yaml

one | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
two | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
three | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

이제 서버 3개를 설정이 다 잘 되었습니다.

## `set_up_init.yml`

이 파일은 아래와 같은 이름으로 라즈베리파이 2개를 현재 자신이 가지고 있는 IP를 고정 IP로 변경한 다음, 이것으로 자동적으로 연결하게 설정한다.

```ini
[all]
1.local
2.local
```


- [ansible.builtin.debug – Print statements during execution — Ansible Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html)