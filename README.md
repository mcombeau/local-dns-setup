# Local DNS Setup

Sets up PiHole, Unbound and Keepalived using Ansible for a secure and private local DNS.

- [Pi-Hole](https://pi-hole.net/) is a network-wide ad-and-malware-blocker, that drops DNS queries for domains known to disseminate ads and malware, preventing their resolution on your network.
- [Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) is a recursive DNS resolver, which is configured here for IPv4 and IPv6 with security measures like DNSSEC verification. It runs independently and does not forward queries to third-parties.

This project can install both of these services on a Raspberry Pi or any Linux machine on your local network (as this is meant for local network use only). The services are installed directly on the machine to avoid the overhead of docker containerization.

## Dependencies

This project uses:

- [Taskfile](https://taskfile.dev/), a more modern type of Makefile: see installation instructions here: [Taskfile installation](https://taskfile.dev/installation/)
- [Ansible](https://www.ansible.com/) to automate the installations to remote hosts via SSH: see installation instructions here: [Ansible installation](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

## Prerequisites

Before running the Ansible playbook, set a password for the Pi-Hole admin dashboard in an .env file (you can copy the `env.example` file and change the password there). The environment variable must be named:

```bash
PIHOLE_WEB_PASSWORD=your_secure_password_here
```

You must also specify the hosts on which to install Pi-Hole and Unbound. For this, edit the `inventories/hosts.yml` file:

```yml
all:
  hosts:
    host_ip:
    another_host_ip:
```

You can indicate as many IP addresses and local domains as you'd like, Ansible will take care of running the install on all of them.

Before running the playbook, you should have your SSH key configured on the hosts you specify here. Otherwise, installation will fail.

## Deployment

When the above requirements are met, you can run the playbook in full:

```bash
task deploy
```

If you want to run the Unbound or Pi-Hole roles separately, you can specify which to deploy:

```bash
task deploy:unbound
task deploy:pihole
```

Every few months, run `task deploy:unbound` again, this will force update the root hints file it needs to start its DNS queries.

Once the installation is done, you can access the Pi-Hole dashboard by visiting `http://host_ip/admin`.

## Configuration

You can find configuration settings for both Unbound and Pi-Hole in `group_vars/all.yml`.

By default, Pi-Hole will listen on port 53 of the machine for DNS queries and will forward them to Unbound running on port 5335.

If you wish to make this the primary DNS resolver on your network, you will have to go into your router and set the DNS servers in your DHCP settings to the host IPs where you installed Pi-Hole and Unbound.

## Testing

To test this is working, take the domain name of one of the hosts you've run the script on and run a `dig` on it:

```bash
dig @host_ip example.org
```

You should get a valid response.

You can also test a domain whose DNSSEC is invalid:

```bash
dig @host_ip dnssec-failed.org
```

This should return a SERVFAIL response.

A domain blocked by Pi-Hole should return a dummy response of 0.0.0.0 (you can test any domain in the [default blocklist](https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts)):

```bash
dig @host_ip ad-assets.futurecdn.net
```

---
Made by [mcombeau](https://miacombeau.com)
