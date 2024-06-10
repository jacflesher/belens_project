## Automating a Script in a Linux VM

### Prerequisites

1. VirtualBox installed (https://www.virtualbox.org/wiki/Downloads)

1. Debian ISO image downloaded (https://www.debian.org/download)

1. A Script that you want to run on a schedule, or use example below:
    ```bash
    #!/bin/bash 
    echo "$(date)" >> /tmp/scheduled_date_stamps.txt  # add a line with date stamp to temp file
    if [[ "$(wc -l < /tmp/scheduled_date_stamps.txt)" -gt 10 ]]; then  # if the file has more than ten lines...
        tail -n 10 > /tmp/scheduled_date_stamps_2.txt  # read the last ten lines and store them in a separate file, and then...
        mv /tmp/scheduled_date_stamps_2.txt /tmp/scheduled_date_stamps.txt  # mv to rename the file to the original name, thus replacing the original file with the new one
    fi
    ```

### Provision the new VM in VirtualBox using the ISO image

    1. make sure to change network adapter to bridged so that the VM gets its own IP

### Configure the VM

    1. Update apt and Install sudo
    ```bash
    su - root
    apt-get update
    apt-get install sudo
    ```

    1. While still root, add yourself to sudoers list
    ```bash
    tee -a /etc/sudoers <<< "user1 ALL=(ALL:ALL) ALL"
    ```
    (Where "user1" is the user ID to give sudo access)

    1. Exit root session
    ```bash
    exit
    ```

    1. Test sudo access of your user ID
    ```bash
    sudo su -
    ```

    1. Install all other packages
    ```
    sudo apt-get install -y curl vim cron ssh ldap-utils git net-tools dnsutils jq
    ```

    1. Configure SSL certs
    ```
    ssh-keygen
    ```

    1. Using scp, get the private key to your local, or if the key was created in local, scp the public cert to the vm and then 
    
    from local
    ```
    scp ~/.ssh/id_rsa.pub user1@1.2.3.4:~/
    ```

    log into vm and then add public key to authorized keys file
    ```
    ssh user1@1.2.3.4
    cat id_rsa.pub > ~/.ssh/authorized_keys
    ```

    1. Build local config file at ~/.ssh/config
    ```
    Host debian
        Hostname 1.2.3.4
        User user1
        IdentityFile ~/.ssh/id_rsa
        ForwardAgent
    ```

    1. Test that you can log into the vm without entering password, just using your ssl certs
    ```
    ssh user1@1.2.3.4
    ```

    1. If successful, at this point we can shut down the vm and set up headless mode


### Configure VM headless mode

    1. Find the name of the VM
    ```
    C:\Users> cd "%programfiles%/Oracle/VirtualBox"
    C:\Program Files\Oracle\VirtualBox> vboxmanage list vms
    "debian01" {f0141d2a-1dd3-4f54-856c-d3dfe1879d82}
    ```

    1. Start the VM as headless so that it runs in the background and you don't need to keep virtualbox open all the time
    ```
    C:\Program Files\Oracle\VirtualBox> VBoxManage startvm "debian01" --type headless
    ```

    1. Now you can log in using SSH again
    ```
    C:\Program Files\Oracle\VirtualBox> ssh user1@1.2.3.4
    ```

    1. To stop the VM, you can log into the VM and do 
    ```
    sudo halt
    ```

    or you can stop the VM using VBoxManage CLI
    ```
    C:\Users> cd "%programfiles%/Oracle/VirtualBox"
    C:\Program Files\Oracle\VirtualBox> VBoxManage controlvm "debian01" poweroff
    ```



