pipeline {

  environment {
    PATH = "/var/lib/jenkins/.local/bin:/usr/local/bin:/usr/bin:/bin"
  }

  agent any

  stages {
    stage('Setup Local Environment') {
      steps {
        echo '--RUNNING LOCAL ENVIORNMENT --'
        sh ''' #!/bin/bash 
        sudo apt-get update -y
        sudo apt-get install python3-dev libffi-dev gcc libssl-dev docker -y
        sudo apt install python3-pip -y
        sudo apt install python3.10-venv -y
        sudo python3 -m venv local
        . local/bin/activate
        ''' 
      }
    }

    stage('INSTALLING PIP') {
      steps {
        echo '--INSTALLING PIP --'
        sh ''' #!/bin/bash 
        pip install -U pip
        ''' 
      }
    }
    stage('INSTALLING Ansible') {
      steps {
        echo '--INSTALLING Ansible --'
        sh ''' #!/bin/bash 
        pip install 'ansible-core>=2.16,<2.17.99'
        ''' 
      }
    }  
    
    stage('INSTALLING Kolla Ansible') {
      steps {
        echo '--INSTALLING Kolla Ansible --'
        sh '''#!/bin/bash 
        pip install git+https://opendev.org/openstack/kolla-ansible@stable/2024.2
        kolla-ansible install-deps
        ''' 
      }
    } 

    stage('Preparing Infrastructure') {
      steps {
        echo '--Preparing Infrastructure Files Structure --'
        sh ''' #!/bin/bash 
        sudo mkdir -p /etc/kolla
        sudo cp -r /usr/local/share/kolla-ansible/etc_examples/kolla/* /etc/kolla/ 
        sudo cp -r /usr/local/share/kolla-ansible/ansible/inventory/* /etc/kolla/ 
	sudo chown -R $USER:$USER /etc/kolla
        sed -i 's/^#kolla_base_distro:.ls*/kolla_base_distro: "ubuntu"/g' /etc/kolla/globals.yml
        sed -i 's/^#enable_haproxy:.*/enable_haproxy: "no"/g' /etc/kolla/globals.yml
        sed -i 's/^#network_interface:.*/network_interface: "eth0"/g' /etc/kolla/globals.yml
        sed -i 's/^#neutron_external_interface:.*/neutron_external_interface: "eth1"/g' /etc/kolla/globals.yml
        sed -i 's/^#kolla_internal_vip_address:.*/kolla_internal_vip_address: "10.0.2.15"/g' /etc/kolla/globals.yml
        ''' 
      }
    }

    stage('Secrets Setup') {
      steps {
        echo '--Generating OpenStack Services Secrets --'
        sh '''#!/bin/bash 
            kolla-genpwd -p /etc/kolla/passwords.yml
           ''' 
      }
    }

    stage('Boostrap Servers') {
      steps {
        echo '--Running Ansible Kolla Boostrap Server Script --'
        sh '''#!/bin/bash 
             kolla-ansible bootstrap-servers -i /etc/kolla/all-in-one
           ''' 
      }
    }

    stage('Infrastructure Pre-Checks') {
      steps {
        echo '--Running Ansible Kolla Prechecks Script --'
        sh '''#!/bin/bash 
            kolla-ansible prechecks -i /etc/kolla/all-in-one
           ''' 
      }
    }

    stage('Deploy Infrastructure') {
      steps {
        echo '--Running Ansible Kolla Prechecks Script --'
        sh '''#!/bin/bash 
        kolla-ansible deploy -i /etc/kolla/all-in-one
        ''' 
      }
    }
  }
}
