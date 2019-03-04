###########################################
# Ansible Docker for Raspberry Pi         #
#   REF: https://docs.ansible.com/ansible # 
###########################################

###############################################################################
# Docker build
time docker build --no-cache -t ernestgwilsonii/docker-raspberry-pi-ansible:2.7.8 -f Dockerfile.armhf .
docker images

# Verify 
docker run --rm ernestgwilsonii/docker-raspberry-pi-ansible:2.7.8 ansible --version

# Upload to Docker Hub
docker login
docker push ernestgwilsonii/docker-raspberry-pi-ansible:2.7.8
###############################################################################


###############################################################################
# First time setup #
####################
# Create bind mounted directory
sudo mkdir -p /opt/ansible
sudo ln -s /opt/ansible /etc/ansible
mkdir -p /etc/ansible
mkdir -p /etc/ansible/files
mkdir -p /etc/ansible/group_vars
mkdir -p /etc/ansible/host_vars
mkdir -p /etc/ansible/templates
mkdir -p /etc/ansible/vault
# Create a starting /etc/ansible/hosts file
echo "[raspberrypi]" >> /etc/ansible/hosts
echo "localhost ansible_host=127.0.0.1" >> /etc/ansible/hosts
echo " " >> /etc/ansible/hosts
# Create a starting /etc/ansible/vault/vault_pass.txt file
echo "raspberry" >> /etc/ansible/vault/vault_pass.txt
# Create a starting /etc/ansible/group_vars/raspberrypi.yml file
echo 'ansible_ssh_user: "pi"' >> /etc/ansible/group_vars/raspberrypi.yml
echo 'ansible_ssh_pass: "raspberry"' >> /etc/ansible/group_vars/raspberrypi.yml
echo 'ansible_ssh_port: "22"' >> /etc/ansible/group_vars/raspberrypi.yml
echo 'ansible_connection: "ssh"' >> /etc/ansible/group_vars/raspberrypi.yml
# Create a starting /etc/ansible/host_vars/localhost.yml file
echo 'ansible_ssh_user: "pi"' >> /etc/ansible/host_vars/localhost.yml
echo 'ansible_ssh_pass: "raspberry"' >> /etc/ansible/host_vars/localhost.yml
echo 'ansible_ssh_port: "22"' >> /etc/ansible/host_vars/localhost.yml
echo 'ansible_connection: "ssh"' >> /etc/ansible/host_vars/localhost.yml
# Create a default starting /etc/ansible/ansible.cfg
echo "[defaults]" > /etc/ansible/ansible.cfg
echo "inventory=/etc/ansible/hosts" >> /etc/ansible/ansible.cfg
echo "host_key_checking=False" >> /etc/ansible/ansible.cfg
echo "retry_files_enabled=False" >> /etc/ansible/ansible.cfg
echo "forks=100" >> /etc/ansible/ansible.cfg
# Add Ansible section to /root/.bashrc
echo " " >> /root/.bashrc
echo "# Ansible #" >> /root/.bashrc
echo "###########" >> /root/.bashrc
# Add ANSIBLE_HOST_KEY_CHECKING=False variable to .bashrc
echo "export ANSIBLE_HOST_KEY_CHECKING=False" >> /root/.bashrc
# Add ANSIBLE_VAULT_PASSWORD_FILE=/etc/ansible/vault/vault_pass.txt variable to .bashrc
echo "export ANSIBLE_VAULT_PASSWORD_FILE=/etc/ansible/vault/vault_pass.txt" >> /root/.bashrc
echo " " >> /root/.bashrc


# Pull down any additional starter playbooks
cd /etc/ansible
wget https://raw.githubusercontent.com/ernestgwilsonii/ansible/master/RaspberryPi_Raspbian-Apply-OS-Updates-playbook.yml
wget https://raw.githubusercontent.com/ernestgwilsonii/ansible/master/RaspberryPi_Raspbian-Install-Docker-playbook.yml


# Setup an "ansible" alias for BASH
#alias 'ansible=docker run --rm -v /etc/ansible:/etc/ansible ernestgwilsonii/docker-raspberry-pi-ansible:2.7.8 ansible'
alias 'ansible=docker run --rm ernestgwilsonii/docker-raspberry-pi-ansible:2.7.8 ansible'


# Test
ansible --version


##########
# Deploy #
##########
# Deploy the stack into a Docker Swarm
docker stack deploy -c docker-compose.yml ansible
# docker stack rm ansible

# Verify
docker service ls | grep ansible
docker service logs -f ansible

###############################################################################
