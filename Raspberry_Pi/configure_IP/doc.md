# 라즈베리파이의 IP를 변경하기

이 글에서는 **ansible**(이하 앤서블)을 이용하여 라즈베리파이의 IP를 변경해보려고 합니다. 물론 파이 한 개를 바꾸는 것은 엔서블을 가지고 하는 것보다 수작업으로 하는 것이 편할 수 있지만, 여러대에 OS를 설치하고, 패키지를 업데이트하고, 네트웍을 손보고, 특정 패키지를 설치하려면, 수작업으로 하는 것은 지루하고 설치할 때 오타가 발생하여 문제가 발생하기도 쉽습니다. 엔서블을 가지고 이런 번거로운 작업을 실행할 수 있다면, 아주 편안하게 서버 작업을 하실 수 있습니다. 우선 서버 한 대부터 시작하여 세 대까지 작업을 확대해보도록 하겠습니다.

현재 라즈베리파이를 설치하는 가장 쉬운 방법은 **Raspberry Pi Imager**을 이용하는 것입니다. 설치하는 방법과 사용 방법을 잘 모르신다면, 다음 링크를 참고하세요.

[How to use Raspberry Pi Imager | Install Raspberry Pi OS to your Raspberry Pi (Raspbian) - YouTube](https://www.youtube.com/watch?v=ntaXWS8Lk34)

앞의 동영상에도 언급되고 있지만, Raspberry Pi Imager v1.6부터는 ‘Ctrl-Shift-X’이라는 단축키를 가지고 advanced options을 설정할 수 있습니다. 이 옵션(options)에는 다음과 같은 것이 들어 있습니다. 이 옵션에 들어 있는 것들이 라즈베리파이의 설정을 쉽게 할 수 있도록 만들어 줍니다.

- Set hostname
- Enable SSH
- Configure wifi
- Set locale settings

여기서 라즈베리파이를 유선랜으로 연결하여 서버로 사용할 때 필요한 옵션은 '호스트 이름 설정(Set hostname)', 'SSH 활성화(Enable SSH)'입니다. 원래는 이미지 파일을 다운 받아서 메모리 카드에 넣은 다음, 이를 라즈베리파이에 설치한 다음, 호스트 이름 설정을 설정하고 SSH 활성화를 해야했는데, 이제는 Raspberry Pi Imager에서 바로 설정한 다음, 이미지 파일을 다운 받아서 메모리 카드에 넣은 후 이를 라즈베리파이에 설치하면 설정이 바로 적용됩니다.

여기서는 호스트 이름은 `raspberrypi.local`으로 설정하고 SSH 활성화를 체크한 다음 비밀번호(password)를 `raspberry`을 설정하였다고 가정하고 글을 쓰겠습니다.

설치가 다 완료되었으면, 아래 명령어를 실행하여 확인합니다.

```bash
ansible all -m ping  --ask-pass --user=pi --inventory 'raspberrypi.local,'
```

설치가 잘 되었으면, 다음과 같은 결과가 나올 것입니다. `SSH password:`는 앞에서 `raspberry`으로 설정했습니다.

```bash
❯ ansible all -m ping  --ask-pass --user=pi --inventory 'raspberrypi.local,'
SSH password: 
raspberrypi.local | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

만약 아래와 같은 에러가 나오는 경우가 있습니다.

```bash
raspberrypi.local | UNREACHABLE! => {
    "changed": false,
    "msg": "Failed to connect to the host via ssh: ssh: Could not resolve hostname raspberrypi.local: nodename nor servname provided, or not known",
    "unreachable": true
}
```

이건 `raspberrypi.local`으로 다른 라즈베리 파이를 설정한 경우가 있으면 발생하는 에러입니다. 아래와 같이 ssh 키를 정리하면 됩니다.

```bash
ssh-keygen -R raspberrypi.local
```

그러면 `host.yml`이라는 파일을 만들어서, 다음 내용을 입력하면 더 쉽게 엔서블을 사용해 봅시다.

```yml
all:
  hosts:
    raspberrypi:
      ansible_host: raspberrypi.local
  vars:
    ansible_python_interpreter: /usr/bin/python3
    ansible_user: pi
    ansible_ssh_pass: raspberry
```

아래와 같은 코드로 앞에서 한 것보다 단순하게 같은 작업을 할 수 있게 됩니다.

```bash
ansible all -m ping -i host.yml
```

`host.yaml`에 많은 정보를 입력했기 때문에(특히 SSH password:)을 저장했기 때문에 직접 입력하지 않아도 됩니다. 결과는 아래와 같습니다. 여기까지 작동한다면 엔서블을 사용하기 위한 에비 작업은 끝난 것입니다.

```bash
❯ ansible all -m ping -i host.yml
raspberrypi | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

기본적으로 라즈베리파이를 설치하면, 기본적으로 DHCP 즉[동적 호스트 구성 프로토콜](https://ko.wikipedia.org/wiki/동적_호스트_구성_프로토콜)로 네트웍을 구성합니다. 그러나 라즈베리파이로 서버를 구성하려면 DHCP으로 구성된 동적(dynamic) IP를 사용하는 것보다. 고정된 정적(static) IP를 하는 것이 중요합니다. 라즈베리파이는 `/etc/dhcpcd.conf`을 이용하여 Static IP address를 구성할 수 있습니다. 자세한 내용은 아래 링크를 참고하세요.

- [TCP/IP networking - Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/configuration/tcpip/)

현재 `/etc/dhcpcd.conf`을 살펴보면, 아래와 같이 `Example static IP configuration:` 부분이 모두 주석 처리되어 있는, 즉 문장 앞에 모두 `#`이 있는 것을 확인하실수 있습니다.

```bash
❯ ansible all -a 'cat /etc/dhcpcd.conf' -i host.yml
raspberrypi | CHANGED | rc=0 >>
# A sample configuration for dhcpcd.
# See dhcpcd.conf(5) for details.

# Allow users of this group to interact with dhcpcd via the control socket.
#controlgroup wheel

# Inform the DHCP server of our hostname for DDNS.
hostname

# Use the hardware address of the interface for the Client ID.
clientid
# or
# Use the same DUID + IAID as set in DHCPv6 for DHCPv4 ClientID as per RFC4361.
# Some non-RFC compliant DHCP servers do not reply with this set.
# In this case, comment out duid and enable clientid above.
#duid

# Persist interface configuration when dhcpcd exits.
persistent

# Rapid commit support.
# Safe to enable by default because it requires the equivalent option set
# on the server to actually work.
option rapid_commit

# A list of options to request from the DHCP server.
option domain_name_servers, domain_name, domain_search, host_name
option classless_static_routes
# Respect the network MTU. This is applied to DHCP routes.
option interface_mtu

# Most distributions have NTP support.
#option ntp_servers

# A ServerID is required by RFC2131.
require dhcp_server_identifier

# Generate SLAAC address using the Hardware Address of the interface
#slaac hwaddr
# OR generate Stable Private IPv6 Addresses based from the DUID
slaac private

# Example static IP configuration:
#interface eth0
#static ip_address=192.168.0.10/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
#static routers=192.168.0.1
#static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1

# It is possible to fall back to a static IP if DHCP fails:
# define static profile
#profile static_eth0
#static ip_address=192.168.1.23/24
#static routers=192.168.1.1
#static domain_name_servers=192.168.1.1

# fallback to static profile on eth0
#interface eth0
#fallback static_eth0
```

위에서 살펴본 `/etc/dhcpcd.conf` 파일에서 적절한 `ip_address`, `static routers`, `static domain_name_servers` 주소를 가지고 아래 부분의 문장 앞에 있는 `#`이 을 제거해 주석 처리된 것을 해체하거나 파일 맨 아래에 새로 작성해서 추가하면 라즈베리파이를 고정 아이피(static IP)로 바꿀 수 있습니다. 여기서 자신이 원하는 주소로 바꾸시면 됩니다.

```bash
interface eth0
static ip_address=192.168.1.23/24
static routers=192.168.1.1
static domain_name_servers=192.168.1.1
```

우선 현재 라즈베리파이가 어떻게 네트웍이 구성되어 있는지 확인해 보겠습니다. 기본적으로 엔서블은 자체적으로 현재 작업하고 있는 서버의 정보를 가지고 있습니다. `ansible_default_ipv4.address`은 현재 `raspberrypi.local`이 사용하고 있는 IP 주소 정보를 가지고 있죠. 만약 호스트 네임을 가지고 해당 호스트가 사용하고 있는 IP 주소 정보를 확인하려면 `hostvars[inventory_hostname]['ansible_default_ipv4']['address']`과 같이 하면 됩니다. 당연히 해당 서버가 사용하고 있는 게이드웨이는 `hostvars[inventory_hostname]['ansible_default_ipv4']['gateway']`를 가지고 알 수 있습니다. 이런 것 내용을 가지고 다음과 같은 코드를 작성해 `temp.yml` 파일에 저장합니다. 참고로 아래 내용에서는 앞의 정보를 변수에 저장해서 사용하고 있습니다. 엔서블에서도 변수를 사용하고 있습니다!

```yml
---
- hosts: all
  user: all
  vars:
    Gateway: " {{hostvars[inventory_hostname]['ansible_default_ipv4']['gateway']}}"
    IP_address: "{{ansible_default_ipv4.address}}"
    ip_address: "{{hostvars[inventory_hostname]['ansible_default_ipv4']['address']}}"
  tasks:
    - name: 현재 IP_address 정보
      ansible.builtin.debug:
        msg: "msg: System {{ inventory_hostname }} has gateway {{ IP_address }}"
    - name: 현재 gateway 정보
      ansible.builtin.debug:
        msg: "msg: System {{ inventory_hostname }} has gateway {{ ip_address }}"
    - name: 현재 gateway 정보
      ansible.builtin.debug:
        msg: "msg: System {{ inventory_hostname }} has gateway {{ Gateway }}"
```

그러면 다음과 같이 해당 서버가 사용하고 있는 정보를 확인할 수 있습니다.

```yml
❯ ansible-playbook temp.yml -i host.yml

PLAY [all] ***********************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************
ok: [raspberrypi]

TASK [현재 IP_address 정보] **********************************************************************************************************************
ok: [raspberrypi] => {
    "msg": "msg: System raspberrypi has gateway 192.168.2.48"
}

TASK [현재 gateway 정보] *************************************************************************************************************************
ok: [raspberrypi] => {
    "msg": "msg: System raspberrypi has gateway 192.168.2.48"
}

TASK [현재 gateway 정보] *************************************************************************************************************************
ok: [raspberrypi] => {
    "msg": "msg: System raspberrypi has gateway  192.168.2.1"
}

PLAY RECAP ***********************************************************************************************************************************
raspberrypi                : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```


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