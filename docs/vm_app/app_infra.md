# Deploy Infrastructure VMs for Application

In this section we will deploy four VMs. Deploy a production-like three-tier application stack on Nutanix AHV using both `Visual Studio Code (VSCode)` and `OpenTofu`:

- 1 x MySQL DB VM (v4 API cURL) - Persistent data storage
- 2 x Frontend Wordpress App VM (Prism UI)
- 1 x HAProxy VM - Load balancer frontend


## Create Infrastructure VMs

1. Open `VSCode`, Go to File -> **New Window** :material-dock-window:, Click on **Open Folder** :material-folder-open: and create new workspace folder (i.e., ``/home/ubuntu``).

2. In `VSCode` Explorer pane, Click on **New Folder** :material-folder-plus-outline: and name it: ``tofu`

3. In the ``tofu`` folder, click on **New File** :material-file-plus-outline: with the following name
  
    ```bash
    cloud-init.yaml.tpl
    ```

4. Paste the following contents inside the file: 
   
    !!! warning
        
        Make sure to paste the contents of the public key created inside the file in the highlighted line

    === ":octicons-file-code-16: cloud-init.yaml.tpl"

        ```yaml hl_lines="26" title=""
        #cloud-config
        hostname: ${hostname}
        fqdn: ${hostname}.local
        manage_etc_hosts: true
        preserve_hostname: false
        package_update: true
        package_upgrade: true
        package_reboot_if_required: true
        packages:
          - open-iscsi
          - nfs-common
        runcmd:
          - hostnamectl set-hostname ${hostname}
          - systemctl stop ufw && systemctl disable ufw
          - eject
          - reboot
        users:
          - default
          - name: ubuntu
            groups: sudo
            shell: /bin/bash
            sudo:
              - 'ALL=(ALL) NOPASSWD:ALL'
            ssh-authorized-keys:
            - ssh-rsa AAAAB3Nxxxxx # (1)
        ```

        1. :material-fountain-pen-tip: Copy and paste the contents of your ``~/.ssh/id_rsa.pub`` file or any public key file that you wish to use.
    
              ---
    
              If you are using a Mac, the command ``pbcopy``can be used to copy the contents of a file to clipboard.
    
              ```bash
              cat ~/.ssh/id_rsa.pub | tr -d '\n' | pbcopy
              ```
    
              ++cmd+"v"++ will paste the contents of clipboard to the console.
    
        !!!warning
              If needed, make sure to update the target `hostname` and copy / paste the value of the RSA public key in the ``cloudinit.yaml`` file.

5. In `VSCode` Explorer, within the ``jumphost-vm`` folder, click on **New File** :material-file-plus-outline: and create a config file with the following name:

    ```bash
    config.yaml
    ```
    
    !!! warning

        Update Nutanix environment access details along with any infrastructure VM configurations. See example file for details

    === ":octicons-file-code-16: Template file"

          ```yaml hl_lines="11 16 17 25 26 34 35"
          nutanix:
            endpoint: "_pc_endpoint_ip_fqdn"
            user: "admin"
            password: "XXXXX_"
          
          infrastructure:
            cluster_name: "_pe_cluster_name"
            subnet_name: "_pe_subnet_name"
          
          vm_defaults:
            image_name: "_ubuntu_image_name" # (1)
          
          roles:
            fe:
              count: 2
              name_prefix: "fe_your_username"
              hostname_prefix: "fe_your_username""
              num_vcpus_per_socket: 1
              num_sockets: 2
              memory_size_mib: 4096
              disk_size_mib: 102400
          
            db:
              count: 1
              name_prefix: "db_your_username"
              hostname_prefix: "db_your_username"
              num_vcpus_per_socket: 1
              num_sockets: 4
              memory_size_mib: 4096
              disk_size_mib: 102400
          
            haproxy:
              count: 1
              name_prefix: "haproxy_your_username"
              hostname_prefix: "haproxy_your_username"
              num_vcpus_per_socket: 1
              num_sockets: 2
              memory_size_mib: 4096
              disk_size_mib: 102400
          ```

          1. Check Images in PC for ubuntu image name that was previously uploaded

    === ":octicons-file-code-16: Example file"

          ```yaml  hl_lines="11 16 17 25 26 34 35"
          nutanix:
            endpoint: "pc.example.com"
            user: "admin"
            password: "XXXXXX_"
          
          infrastructure:
            cluster_name: "pe"
            subnet_name: "primary"
          
          vm_defaults:
            image_name: "ubuntu-24.04" # (1)
          
          roles:
            fe:
              count: 2
              name_prefix: "fe-user01"
              hostname_prefix: "fe-user01"
              num_vcpus_per_socket: 1
              num_sockets: 2
              memory_size_mib: 4096
              disk_size_mib: 102400
          
            db:
              count: 1
              name_prefix: "db-user01"
              hostname_prefix: "db-user01"
              num_vcpus_per_socket: 1
              num_sockets: 4
              memory_size_mib: 4096
              disk_size_mib: 102400
          
            haproxy:
              count: 1
              name_prefix: "haproxy-user01"
              hostname_prefix: "haproxy-user01"
              num_vcpus_per_socket: 1
              num_sockets: 2
              memory_size_mib: 4096
              disk_size_mib: 102400
          ```

          2. Check Images in PC for ubuntu image name that was previously uploaded
   
6. In `VSCode` Explorer pane, navigate to the ``jumphost-vm`` folder, click on **New File** :material-file-plus-outline: and create a opentofu manifest file with the following name:

    ```bash
    vm.tf
    ```

    with the following content:
    
    === ":octicons-file-code-16: vm.tf"

    ```json
    ############################
    # Locals
    ############################
    
    locals {
      config = yamldecode(file("${path.module}/config.yaml"))
    
      # Expand roles into individual VM definitions
      vms = flatten([
        for role_name, role in local.config.roles : [
          for i in range(role.count) : {
            role                  = role_name
            index                 = i + 1
            name                  = format("%s-%02d", role.name_prefix, i + 1)
            hostname              = format("%s-%02d", role.hostname_prefix, i + 1)
            num_vcpus_per_socket  = role.num_vcpus_per_socket
            num_sockets           = role.num_sockets
            memory_size_mib       = role.memory_size_mib
            disk_size_mib         = role.disk_size_mib
          }
        ]
      ])
    
      vm_map = {
        for vm in local.vms :
        "${vm.role}-${vm.index}" => vm
      }
    }
    
    ############################
    # Provider
    ############################

    provider "nutanix" {
      endpoint = local.config.nutanix.endpoint
      username = local.config.nutanix.user
      password = local.config.nutanix.password
      insecure = true
    }
    
    data "nutanix_cluster" "cluster" {
      name = local.config.infrastructure.cluster_name
    }
    
    data "nutanix_subnet" "subnet" {
      subnet_name = local.config.infrastructure.subnet_name
    }
    
    ############################
    # Image lookup
    ############################
    
    data "nutanix_image" "ubuntu" {
      image_name = local.config.vm_defaults.image_name
    }
    
    ############################
    # VMs
    ############################
    
    resource "nutanix_virtual_machine" "vm" {
      for_each = local.vm_map
    
      name         = each.value.name
      cluster_uuid = data.nutanix_cluster.cluster.id
    
      num_vcpus_per_socket = each.value.num_vcpus_per_socket
      num_sockets          = each.value.num_sockets
      memory_size_mib      = each.value.memory_size_mib
    
      disk_list {
        data_source_reference = {
          kind = "image"
          uuid = data.nutanix_image.ubuntu.id
        }
        disk_size_mib = each.value.disk_size_mib
      }
    
      nic_list {
        subnet_uuid = data.nutanix_subnet.subnet.id
      }
    
      guest_customization_cloud_init_user_data = base64encode(
        templatefile("${path.module}/cloud-init.yaml.tpl", {
          hostname = each.value.hostname
        })
      )
    
      depends_on = [data.nutanix_image.ubuntu]
    }
    
    
    ############################
    # Outputs
    ############################
    
    output "vm_ips_by_role" {
      description = "IP addresses grouped by role"
      value = {
        for role in distinct([for v in local.vms : v.role]) :
        role => [
          for k, vm in nutanix_virtual_machine.vm :
          {
            name = vm.name
            ips  = flatten([
              for nic in vm.nic_list :
              nic.ip_endpoint_list[*].ip
            ])
          }
          if local.vm_map[k].role == role
        ]
      }
    }
    ```

7. Open a terminal within `VSCode`, **Terminal > New Terminal** :octicons-terminal-16:
8. Make sure to change to ``tofu`` directory
   
    ```bash
    cd $HOME/tofu
    ```

9. Initialize and Validate your tofu code

    ```bash
    tofu init
    ```

    ```bash
    tofu validate
    ```

10. Plan your tofu code to create Jump Host VM
  
    ```bash
    tofu plan 
    ```

11. Apply your tofu code to create Jump Host VM
  
    ```bash
    tofu apply 
    ```

    Type ``yes`` to confirm

12. Obtain the IP address of the `Jump Host` VM from the Tofu output
  
    ``` { .bash .no-copy }
    vm_ips_by_role = {
    "db" = [
      {
        "ips" = [
          "10.x.x.124",
        ]
        "name" = "db-user01-01"
      },
    ]
    "fe" = [
      {
        "ips" = [
          "10.x.x.130",
        ]
        "name" = "fe-user01-01"
      },
      {
        "ips" = [
          "10.x.x.131",
        ]
        "name" = "fe-user01-02"
      },
    ]
    "haproxy" = [
      {
        "ips" = [
          "10.x.x.123",
        ]
        "name" = "haproxy-user01-01"
      },
    ]
    ```

13. Run the Terraform state list command to verify what resources have been created

    ``` bash
    tofu state list
    ```

    ``` { .bash .no-copy }
    # Sample output for the above command

    data.nutanix_cluster.cluster              # < This is your existing Prism Element cluster
    data.nutanix_subnet.subnet                # < This is your existing primary subnet
    nutanix_image.machine-image               # < This is the ubuntu image file
    nutanix_virtual_machine.vm["db-1"]        # < This is the db vm
    nutanix_virtual_machine.vm["fe-1"]        # < This is the fe-1 vm
    nutanix_virtual_machine.vm["fe-2"]        # < This is the fe-2 vm
    nutanix_virtual_machine.vm["haproxy-1"]   # < This is the haproxy vm
    ```

14. Validate that all the infrastructure VMs are accessible using **VSCode > Terminal** :octicons-terminal-24:
  
    === "Command"

        ```bash
        ssh -l ubuntu <ip-address-of-db-user01-01>
        ssh -l ubuntu <ip-address-of-fe-user01-01-vm>
        ssh -l ubuntu <ip-address-of-fe-user01-02-vm>
        ssh -l ubuntu <ip-address-of-haproxy-user01-01>
        ``` 

    === "Command Sample"

        ```{ .text .no-copy }
        ssh -l ubuntu 10.x.x.124
        ssh -l ubuntu 10.x.x.130
        ssh -l ubuntu 10.x.x.131
        ssh -l ubuntu 10.x.x.123
        ```     

!!! warning

    Wait for at least 5 minutes for all the VMs to install updates and reboot

    Watch the events and Console in Prism Central.


Now the infrastructure VMs are ready with all the tools to deploy application on this site. 