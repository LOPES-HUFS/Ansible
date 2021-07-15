# 앤서블 개념(Ansible concepts)

<!-- These concepts are common to all uses of Ansible. -->
이 개념들은 앤서블에서 공통적으로 모두 사용하는 것입니다.
<!-- You need to understand them to use Ansible for any kind of automation. -->
여러분이 앤서블로 다양한 자동화(automation)를 사용하기 위해서는 이 개념들을 이해할 필요가 있습니다.
<!-- This basic introduction provides the background you need to follow the rest of the User Guide. -->
이 기본적인 소개는 여러분이 이 유저 가이드 나머지를 따라 이해하기 위해 필요한 배경을 제공합니다.

## 관리자 노드(Control node)

<!-- Any machine with Ansible installed. -->
앤서블이 설치된 머신
<!-- You can run Ansible commands and playbooks by invoking the ansible or ansible-playbook command from any control node. -->
여러분은 앤서블 명령어들와 각본(playbook)들을 앤서블 또는 앤서블-각본 명령어를 어떠한 관리자 노드에서도 호출하여 실행할 수 있습니다.
<!-- You can use any computer that has a Python installation as a control node - laptops, shared desktops, and servers can all run Ansible. -->
파이썬이 설치된 컴퓨터라면 관리자 노드로 사용할 수 있습니다 - 랩탑, 공유된 데스크탑, 그리고 서버 모두 앤서블이 작동합니다.
<!-- However, you cannot use a Windows machine as a control node. You can have multiple control nodes. -->
그러나, 윈도우 머신은 관리자 노드로 사용할 수 없다. 여러분은 멀티플 관리자 모드들을 사용할 수 있다.

## 관리 노드(Managed nodes)

<!-- The network devices (and/or servers) you manage with Ansible. -->
여러분이 앤서블로 관리 네트워크 장치(device)들 (그리고/ 또는 서버들).
<!-- Managed nodes are also sometimes called “hosts”. -->
관리 노드들은 또한 때때로 “호스트(host)들”이라고 불리기도 합니다.
<!-- Ansible is not installed on managed nodes. -->
앤서블은 관리 노드에는 설치되지 않습니다.

## 인벤토리(Inventory)

<!-- A list of managed nodes. -->
관리 노드들의 목록
<!-- An inventory file is also sometimes called a “hostfile”. -->
인벤토리 파일은 또란 "호스트파일(hostfile)"이라고 부르기도 합니다.
<!-- Your inventory can specify information like IP address for each managed node. -->
여러분의 인벤토리는 관리 노드를 위한 각각의 IP와 같은 정보를 기술할 수 있습니다.
An inventory can also organize managed nodes, creating and nesting groups for easier scaling.
인벤토리는 또한 관리 노드들을, 그룹을 마련하고 조직(organize)할 수 있다.
<!-- To learn more about inventory, see the Working with Inventory section. -->
인벤토리에 대하여 더 많이 알려면, 인벤토리로 일하기 절을 보세요.

## 컬렉션(Collection)s

<!-- Collections are a distribution format for Ansible content that can include playbooks, roles, modules, and plugins. -->
컬렉션은 각본(playbook)들과 역할(role)들, 모듈들, 그리고 플러그인들을 포함할 수 있는 앤서블 컨텐츠를 위한 분산 포맷입니다.
<!-- You can install and use collections through Ansible Galaxy. -->
여러분은 앤서블 갤럭시를 통해서 컬렉션을 사용하거나 수집할 수 있습니다.
<!-- To learn more about collections, see Using collections. -->
좀 더 컬렉션을 배우기 위해서는, 컬렉션 사용하기를 보면 됩니다.

## 모듈(Module)s

<!-- The units of code Ansible executes. -->
앤서블 코드 수행 단위
Each module has a particular use, from administering users on a specific type of database to managing VLAN interfaces on a specific type of network device.
각 모듈은 특정 유형의 데이터베이스에서 사용자를 관리하는(administering) 것부터 특정 유형의 네트워크 장치(device)에서 VLAN 인터페이스를 관리하는(managing) 것에 이르기까지 특정 용도를 가지고 있습니다.
You can invoke a single module with a task, or invoke several different modules in a playbook.
여러분은 과제 하나를 가지고 모둘 하나를 호출하거나(invoke), 또는 하나의 각본(playbook)안에서 여러 다른 모듈들을 호출할 수 있습니다.
<!-- Starting in Ansible 2.10, modules are grouped in collections. -->
앤서블 2.10부터, 모듈들은 컬렉션들 안에 모여있게 된다.
For an idea of how many collections Ansible includes, take a look at the Collection Index.
Ansible에 포함된 수많은 컬렉션에 대한 아이디어는, Collection Index를 살펴보십시오.

## 과제(Task)s

<!-- The units of action in Ansible. -->
앤서블에서 행동(action) 단위
<!-- You can execute a single task once with an ad-hoc command. -->
여러분은 에드-훅(ad-hoc) 명령으로 한 번에 한 관제만 수행할 수 있다.

## 각본(Playbook)s

<!-- Ordered lists of tasks, saved so you can run those tasks in that order repeatedly. -->
과제들의 목록을 정렬하고, 그 순서대로 저장하여 여러분은 그 목록 순서대로 반복적으로 이러한 목록 순서대로 과제를 실행할 수 있다.
<!-- Playbooks can include variables as well as tasks. -->
각본은 과제뿐만 아니라 변수들도 포함할 수 있습니다.
<!-- Playbooks are written in YAML and are easy to read, write, share and understand. -->
각본은 YAML 포맷으로 쓰여지기에 읽기도 쓰기도 공유하기도 쉽습니다.
<!-- To learn more about playbooks, see Intro to playbooks. -->
각본에 대해서 더 배우기 위해서는, [각본(playbooks) 소개](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#about-playbooks)를 보세요.
