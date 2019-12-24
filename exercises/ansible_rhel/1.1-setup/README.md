# Exercise 1.1 - Check the Prerequisites

**Read this in other languages**: ![uk](../../../images/uk.png) [English](README.md),  ![japan](../../../images/japan.png)[日本語](README.ja.md).

## Your Lab Environment

In this lab you work in a pre-configured lab environment. You will have access to the following hosts:

| Role                 | Inventory name |
| ---------------------| ---------------|
| Ansible Control Host | ansible        |
| Managed Host 1       | node1          |
| Managed Host 2       | node2          |
| Managed Host 2       | node3          |

## Step 1.1 - Access the Environment

Login to your control host via SSH:

> **Warning**
>
> Replace **11.22.33.44** by your **IP** provided to you.

    ssh root@11.22.33.44

> **Tip**
>
> The password is **ansible**

Most prerequisite tasks have already been done for you:

  - SSH connection and keys are configured

  - `sudo` has been configured on the managed hosts to run commands that require root privileges.
  
Install Ansible on your Ansible controller host:

    [root@ansible ~]#
    wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
    yum install epel-release-latest-7.noarch.rpm -y
    yum install ansible -y
    yum install python-argcomplete -y
    [...]

Check Ansible has been installed correctly

    [root@ansible ~]# ansible --version
    ansible 2.9.1
    [...]

> **Note**
>
> Ansible is keeping configuration management simple. Ansible requires no database or running daemons and can run easily on a laptop. On the managed hosts it needs no running agent.

Log out of the root account again:

    [root@ansible ~]# exit
    logout

> **Note**
>
> In all subsequent exercises you should work as the root user on the control node if not explicitly told differently.

## Step 1.2 - Working the Labs

You might have guessed by now this lab is pretty commandline-centric…​ :-)

  - Don’t type everything manually, use copy & paste from the browser when appropriate. But stop to think and understand.

  - All exercises were prepared using **Vim**, but we understand not everybody loves it. Feel free to use alternative editors.

----

[Click here to return to the Ansible for Red Hat Enterprise Linux Workshop](../README.md#section-1---ansible-engine-exercises)
