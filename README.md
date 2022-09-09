This project is based on ansible for installing an on-premise Kubernetes cluster all in once.
For api server high availability haproxy and keepalived are also installed.
Ingress (nginx and istio based), metallb are also installed.
You have to make necessary changes (set IPs, registry location etc.) inside allvars.yaml and ansible_hosts files before installation

For running remote tasks on servers, we use the password assigned to the variable ansible_become_pass
We use vault in order to keep the password encrypted assigned to the variable ansible_become_pass
With the below command, we can encrypt the password, using the password in a_password_file file.
We write the password of the user which we use to connect to the servers using ssh, in a_password_file 
After generating the encrypted password and adding it to the bottom of the allvars.yaml file we can safely delete the a_password_file

paste the output of the below command to the bottom of the allvars.yaml file 
* ansible-vault encrypt_string --vault-password-file a_password_file 'write here the password of the ssh user used to connect to the servers' --name 'ansible_become_pass'

In order to  run our ansible playbook run the below command
When asked for password enter the ssh password you used above
* ansible-playbook kubernetes_cluster_install.yaml -k --ask-vault-pass -vv --extra-vars 'FORCE_KUBE_INSTALL=1' -i ansible_hosts

In order to run the playbook task by task run the below command
* ansible-playbook kubernetes_cluster_install.yaml -k --ask-vault-pass -vv --step --extra-vars 'FORCE_KUBE_INSTALL=1' -i ansible_hosts

In order to run a particular task run the below command
* ansible-playbook kubernetes_cluster_install.yaml -k --ask-vault-pass -vv --tags "install_calico" --extra-vars 'FORCE_KUBE_INSTALL=1' -i ansible_hosts 


'FORCE_KUBE_INSTALL=1' parameter is defined by us in order to force the installation to a previously installed kubernetes cluster.
* We also have some more checks other than this parameter.


** IMPORTANT NOTES ****
- Some of the dashboard components are not installed (seems a bug in ansible) when you run the ansible playbook first time.
  In order to overcome this issue it may be necessary to run the single task again using the task's tag.
  
  * ansible-playbook kubernetes_cluster_install.yaml -k --ask-vault-pass -vv --tags "install_kubernetes_dashboard" --extra-vars 'FORCE_KUBE_INSTALL=1' -i ansible_hosts


- The output of kubeadm init and the dashboard token are written to the files .kubeadm_init_output.txt and .dashboard_admin_token.txt respectively in the master server's home directory
