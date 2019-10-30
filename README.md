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

You need the following packages installed on your Centos 7 workstation to create an configure the RHEL 7.7 VM image.

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

#### Next Steps

1. Download the latest 7.7 (or later) version of [Red Hat Enterprise Linux](https://access.redhat.com/downloads/content/69/ver=/rhel---7/7.7/x86_64/product-software) from the Red Hat Customer Portal. You should download the Red Hat Enterprise Linux KVM Guest Image.
2. Move the image to ```/var/lib/libvirt/images```
3. Create a new VM using virt-manager. Note the following when creating the VM.
   - Import an existing disk image and select the downloaded qcow2 file.
   - Accept or change the memory and CPU settings to your application requirements.
   - Select the Customize configuration before install check box.
   - On the custom configuration dialog box, make sure that virtio is set as the NIC Device model, and then begin the installation.
   - The VM may hang momentarily at the IPV6 eth0 line when booting up. This is normal at this point in the installation.
   - ![Screenshot](images/virtiorhelvm.png)
   - For detailed virt-manager instructions, see [Create the RHEL VM from a RHEL KVM Guest Image](https://access.redhat.com/articles/uploading-rhel-image-to-azure#header11)
4. Shut down the VM when you see the login prompt.
5. From your RHEL system, set up root access to the VM. Use virt-customize to generate a root password for the VM. Make sure to create a strong password.
   ```
   # virt-customize -a <guest-image-path> --root-password password:<password>
   ```
   Example:
   ```
   # virt-customize -a /var/lib/libvirt/images/rhel-server-7.7-x86_64-kvm.qcow2 --root-password password:<password>
   [   0.0] Examining the guest ...
   [ 103.0] Setting a random seed
   [ 103.0] Setting passwords
   [ 112.0] Finishing off
   ```
6. Verify root access by starting the VM and logging in as root.
7. Verify that python-boto and cloud-init are installed on the VM.
   - python-boto is required for Red Hat Cloud Access RHEL VMs on GCP. This package is installed on the Red Hat Enterprise Linux KVM Guest Image. If python-boto is not installed on a custom image, enable the rhel-7-rh-common-rpms repository and install it.
   - cloud-init handles cloud provisioning and is also installed on the KVM Guest image. If you need to install it, the package is available in rhel-7-server-rpms.
   ```
   # rpm -qa | grep python-boto
   # rpm -qa | grep cloud-init
   ```
8. Temporarily enable root password access. Edit /etc/cloud/cloud.cfg and update ssh_pwauth from 0 to 1.
   ```
   ssh_pwauth: 1
   ```

#### Prepare and Import the base GCP image

Complete the following steps to prepare the image for GCP

1. Enter the following command to convert the file. Images uploaded to GCP need to be in raw format and named disk.raw.
   ```
   $ qemu-img convert -f qcow2 <ImageName>.qcow2 -O raw disk.raw
   ```
2. Enter the following command to compress the raw file. Images uploaded to GCP need to be compressed.
   ```
   $ tar -Sczf <ImageName>.tar.gz disk.raw
   ```
3. Import the compressed image to the bucket created earlier.
   ```
   $ gsutil cp <ImageName>.tar.gz gs://<BucketName>/
   ```

#### Create and configure a base GCP instance


   
   