# VPN Relay for HomeBox

Self-hosting your emails, calendars, contacts, etc. is nice, but requires one thing that is sometimes difficult to
obtain: a static - or fixed - IP address. Most ISP (Internet Services Providers) are providing a dynamic IP address,
incompatible with self hosting.

The purpose of this project is to provide at least one static IP address to your
[HomeBox](https://github.com/progmaticltd/homebox), even if you are behind a dynamic IP address or even a 3G/4G internet
connection. To do so, we are using an OpenVPN server installed on a lightweight instance from a cloud providers.
Your files are still stored at home, on your hardware. The VPN access point only relays the internet traffic to your
box, especially the remote connections.

This also easily solves all NAT traversal issues.

## Installation steps

This is a high level view of the steps necessary to create the relay server, in this case on Vultr. The steps are
detailed below.

1. Create an account on Vultr, and get an API key
2. Copy the system-example.yml into system.yml
3. Run the Ansible playbook to create the relay server
4. Create the hosts.yml file with your new server IP address
5. Run the VPN server installation playbook

### Create an account on Vultr

At this time of testing, I am using Vultr as a provider, for specific reasons, essentially the port 25 not blocked by
default. Maybe other services are possible, for instance AWS. At this date, the procedure to unblock the port 25 on
DigitalOcean is long.

The cost for a single instance, with two IP addresses, IPv4 and IPv6 is reasonable, 5$ per month, so 60$ per year. Most
of the time, this will be cheaper than using a professional internet service provider, just to get a static IP address.

https://www.vultr.com/

Once the account created, go to my account, and select the API entry. You should be able to create a key, that looks
like the following string: `RARAKIDPLEHD8673KLSWPOCNBEUKS754KMSE`

You can export it and/or store into the system.yml file. This file is excluded by git. See the
[Ansible documentation](https://docs.ansible.com/ansible/latest/modules/vultr_account_info_module.html) to know where to
store your API key too.

### Create the system.yml file from the example one

```yml

vultr:
  api:
    account: default
    key: RARAKIDPLEHD8673KLSWPOCNBEUKS754KMSE

system:
  name: homebox-relay                      # If you are using dynamic inventory, this name should be in your hosts file too.
  hostname: foxy                           # The host name is not really important, you can put anything you want
  ipv6_enabled: true                       # If you want to use an IPv6 for your box too
  region: London                           # See https://www.vultr.com/features/datacenter-locations/
  os: Debian 10 x64 (buster)               # Perhaps you want to keep this
  plan: 1024 MB RAM,25 GB SSD,1.00 TB BW   # and this too. See Vultr doc to see the possible values
  ssh_keys:                                #
    - name: default                        # these keys will be copied to the /root/.ssh/authorized_keys
      path: /home/andre/.ssh/id_rsa.pub

```

### Create the server

Once the configuration ready, you can run the Ansible playbook:

```sh
cd install
ansible-playbook -v -l localhost playbooks/vultr.yml
```

Once the relay server created, you should have its IP addresses. These addresses will be used for your HomeBox, as
external static IP addresses, even if you are behind a dynamic IP address.


### Create your hosts file

Copy the hosts-example.yml into hosts.yml, and set the IP address of your new server. For instance:

```yaml

# This configuration should contains two hosts:
# The relay server, and your local homebox IP address
all:
  hosts:
    homebox-relay:
      ansible_host: 12.34.53.78
      ansible_user: root
      ansible_port: 22
    homebox:
      ansible_host: 192.168.64.33
      ansible_user: root
      ansible_port: 22
```

### Run the VPN server installation playbook

Work in progress...
