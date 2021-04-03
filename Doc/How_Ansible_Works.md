# 엔서블을 어떻게 일할까?

[How Ansible Works | Ansible.com](https://www.ansible.com/overview/how-ansible-works)

Ansible is a radically simple IT automation engine that automates cloud provisioning, configuration management, application deployment, intra-service orchestration, and many other IT needs.

Designed for multi-tier deployments since day one, Ansible models your IT infrastructure by describing how all of your systems inter-relate, rather than just managing one system at a time.

It uses no agents and no additional custom security infrastructure, so it's easy to deploy - and most importantly, it uses a very simple language (YAML, in the form of Ansible Playbooks) that allow you to describe your automation jobs in a way that approaches plain English.

On this page, we'll give you a really quick overview so you can see things in context. For more detail, hop over to docs.ansible.com.

## 효율적인 구조(EFFICIENT ARCHITECTURE)

Ansible works by connecting to your nodes and pushing out small programs, called "Ansible modules" to them. -->
앤서블을 여러분의 노드들(nodes)과 연결하고, "앤서블 모듈들(Ansible module)"이라고 부르는 작은 프로그램들 
These programs are written to be resource models of the desired state of the system. 
이런 프로그램들은 시스템의 
Ansible then executes these modules (over SSH by default), and removes them when finished.

Your library of modules can reside on any machine, and there are no servers, daemons, or databases required. Typically you'll work with your favorite terminal program, a text editor, and probably a version control system to keep track of changes to your content.
