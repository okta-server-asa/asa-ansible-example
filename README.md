# asa-ansible-example

This is a sample playbook for enrolling servers  Okta Advanced Server Access (ASA) for enrolling and securing servers with .

In this example, We showcase the full power of Okta and Ansible to automate and secure servers. 

To leverage this example, you must have an Okta ASA administrative account and an Ansible. 
NOTE: If you don’t have an Okta ASA account yet, you can go and [create one here](https://app.scaleft.com/p/signup).


# Setup

Clone this repository:

```
git clone https://github.com/sudobinbash/asa-ansible-example.git
cd asa-ansible-example
```

Edit the `asa-playbook.yml`. on line 3, replace `<hosts>` with the hosts you want to enroll in ASA (line 3):

```
-
  name: Install SFT
  hosts: <hosts>
```

Get An Enrollment in ASA:

1. Access Okta ASA as administrator.
2. Click projects, and then select (or create a project).
3. Click Enrollment > Create Enrollment Token.
4. Enter a name (i.e. ansible-token) and click Submit.
5. Copy the enrollment token

![ASA: Enrollment token](img/asa-get-token.png)

Enroll servers using the ansible-playbook command:

```
ansible-playbook asa-playbook.yml -i <inventory> --extra-vars "asa_enrollment_token=<token>"
```

Replace: 
- `<inventory>`: with your inventory file (or any list of servers)
- `<token>`: with the enrollment token from ASA

Ansible will run and register your servers, and return the following:

![Ansible Summary](img/ansible-summary.png)

In ASA, you will see the servers enrolled in your project:

![ASA: Servers Enrolled](img/asa-list-servers.png)


## Power Tips / FAQ

**Wait wait... what does the asa-playbook.yml do?**

The `asa-playbook.yml` playbook installs the ASA server agent and then enroll your servers into ASA for remote access.

Our playbook supports multiple Linux distros, and We use the server distro family to identify the proper installation tasks:

```
  tasks:     
    #INSTALL
    - name: Install Debian
      include_tasks: asa-samples/install-debian.yml
      when: ansible_facts['os_family'] == "Debian"
    
    - name: Install RedHat
      include_tasks: asa-samples/install-redhat.yml
      when: ansible_facts['os_family'] == "RedHat"
    
    - name: Install Suse
      include_tasks: asa-samples/install-suse.yml
      when: ansible_facts['os_family'] == "Suse"
```

The steps for enrolling the servers are the same across all Linux distros, and listed on the after the installation tasks.

**How do I define the server names listed in ASA? (aka Canonical Names)**

The ASA servers are registered with a canonical name, that can be used by users to access these servers (i.e. `ssh <canonical-name>`). In this example, we are using the Ansible inventory_hostname as the canonical name. You can change the name according to your preferences, using variables and server facts from Ansible.

```
    - name: Creating canonical file (using server names from the inventory)
      copy:
        dest: "/etc/sft/sftd.yaml"
        content: |
          ---
          # CanonicalName: Specifies the name clients should use/see when connecting to this host.
          CanonicalName: "{{ inventory_hostname }}"
      become: yes
```

**Can I use this playbook with other Ansible provisioning modules?**

Yes. You can use this sample with any other module available in [Ansible](https://docs.ansible.com/ansible/latest/collections/index.html) or the Ansible Galaxy. This includes IaaS providers and Hypervisors, such as [AWS](https://docs.ansible.com/ansible/latest/collections/amazon/aws), [VMWare](https://docs.ansible.com/ansible/latest/collections/community/vmware), [GCP](https://docs.ansible.com/ansible/latest/collections/google/cloud), [OpenStack](https://docs.ansible.com/ansible/latest/collections/openstack/cloud), and [Azure](https://docs.ansible.com/ansible/latest/collections/azure/azcollection). To do this, use the inventory plugins available with these modules to capture your inventory and then apply the playbook for your servers.

**Any extra recommendations?**

Absolutely yes. Here are some things to remember:

**Test first before broad rollout:** Make sure you experiment the `asa-playbook.yml` as well as ASA before enrolling all your server fleet in ASA (i.e. set `hosts: all`).

**Securing the enrollment token:** In our example, we assigned the ASA enrollment token directly in the `asa_enrollment_token` variable for testing purposes. For production environments, we recommend using a secret manager, like [Ansible's vault](https://docs.ansible.com/ansible/2.8/user_guide/vault.html). Here's a [cool example from Tristan Fisher](https://gist.github.com/tristanfisher/e5a306144a637dc739e7).


**Securing the Ansible Controller**: Yes yes... you can also secire the Ansible controller with ASA. To do that, just enroll your controller in ASA, using the same playbook:
1. Update your `asa-playbook.yml`, changing the hosts to `local`
2. Run the playbook pointing to your local environment. i.e.: `ansible-playbook  -c local -i localhost asa-playbook.yml --extra-vars "asa_enrollment_token=<local>"`
