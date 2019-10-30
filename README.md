## Creating a custom RHEL 7.7+ Server image on Google Cloud Platform

In this section we'll use the Centos 7 Workstation created in the previous guide [Installing / Configuring GNOME and VNC Server](https://github.com/chainlynx/google-cloud-centos-workstation/blob/master/Installing-Configuring-GNOME.md) to setup a custom RHEL 7.7+ image that can then be used in setting up the nodes for your Openshift Enterprise cluster. 

Google Cloud Platform also provides a default RHEL 7 image, in the available images when creating an instance, however this instance also includes a usage fee of about $43.80/month. Basically this image already comes with a Red Hat subscription and therefore use of this image, even if you update it with your own subscription, is subject to this fee. More information [here](https://console.cloud.google.com/marketplace/details/rhel-cloud/rhel-7).

Therefore, this guide is aimed at creating a custom image, adding support for the Google Cloud Platform, and running an instance of RHEL 7 without incurring this additional fee.

#### Prerequisites

- You must belong to an active GCP project and have sufficient privaledges to create resources in the project.

- To proceed with this guide, you will need a Red Hat subscription, with the ability to access the base images here: 

- **Note:** I also have a RHEL 7.7 image created that can be shared, to avoid building one yourself, and saving you a few steps, however it still requires activation with your subscription on each node you create with the image.

### Getting Started

Complete the following steps to create the base RHEL 7.7 VM image for GCP. Complete this procedure using a RHEL 7 Server, Centos 7 Server, or Fedora. For the purpose of this guide, we'll use the Centos 7 workstation created in the previous steps: [Installing / Configuring GNOME and VNC Server](https://github.com/chainlynx/google-cloud-centos-workstation/blob/master/Installing-Configuring-GNOME.md).

#### Before you Start

You need the following packages on your Centos 7 workstation to create an configure the RHEL 7.7 VM image.

| Package | Repository | Description |
| ------- |:---------- |:----------- |
| libvirt | rhel-7-server-rpms | Open source API, daemon, and management tool for managing platform virtualization |
| virt-manager | rhel-7-server-rpms | A command line and Gnome desktop virtual machine manager used to create and manage KVM virtual machines |
| libguestfs | rhel-7-server-rpms | A library for accessing and modifying virtual machine file systems |
| libguestfs-tools | rhel-7-server-rpms | System administration tools for virtual machines; includes the guestfish utility |

<table>

  <tr bgcolor="#cccccc">
    <th align="left">Package</th>
    <th align="left">Repository</th>
    <th align="left">Description</th>
  </tr>
  <tr>
    <td nowrap="true">libvirt</td>
    <td nowrap="true">rhel-7-server-rpms</td>
    <td>Open source API, daemon, and management tool for managing platform virtualization</td>
  </tr>
  <tr>
    <td nowrap="true">virt-manager</td>
    <td nowrap="true">rhel-7-server-rpms</td>
    <td>A command line and Gnome desktop virtual machine manager used to create and manage KVM virtual machines</td>
  </tr>
  <tr>
    <td nowrap="true">libguestfs</td>
    <td nowrap="true">rhel-7-server-rpms</td>
    <td>A library for accessing and modifying virtual machine file systems</td>
  </tr>
  <tr>
    <td nowrap="true">libguestfs-tools</td>
    <td nowrap="true">rhel-7-server-rpms</td>
    <td>System administration tools for virtual machines; includes the guestfish utility</td>
  </tr>
</table>