---


---

<h1 id="how-to-use-an-ubuntu-cloud-image-in-titaniumopenstack">How to Use an Ubuntu Cloud Image in Titanium/OpenStack</h1>
<p>Author: <a href="mailto:richard.moore@windriver.com">richard.moore@windriver.com</a></p>
<p>Cloud images are much easier to use than installing from scratch and are typically maintained by the Linux distribution publisher, like Canonical for Ubuntu.</p>
<p>This tutorial uses remote clients (delivered through a docker container) but most of the operations can also be done through Horizon.</p>
<h3 id="download-bionic-cloud-image-from-ubuntu-cloud-repo">1 ) Download bionic cloud image from <a href="https://cloud-images.ubuntu.com/bionic/current/">Ubuntu Cloud Repo</a></h3>
<pre><code>wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
</code></pre>
<h3 id="run-the-tic-sdk-container-with-the-remote-clients-skip-this-if-you-already-have-the-remote-clients-from-the-tic-sdk-installed.">2 ) Run the tic-sdk container with the remote clients (skip this if you already have the remote clients from the TiC SDK installed).</h3>
<blockquote>
<p>Note: your host must have docker installed. This works on Windows 7,10/Mac/Linux if docker is installed correctly.</p>
</blockquote>
<h4 id="export-location-of-your-openrc-credentials-files">2.1) export location of your openrc credentials file(s):</h4>
<pre><code>export RC_DIR=/home/myuserid/my_rcfiles/
</code></pre>
<blockquote>
<p>Note: download the credentials file by logging into horizon</p>
</blockquote>
<h4 id="cd-to-the-directory-with-the-cloud-image-and-run-the-tic-sdk-docker-container-again-skip-if-tic--remote-client-tools-are-installed.">2.2) CD to the directory with the cloud image and run the tic-sdk docker container (again, skip if TiC  remote client tools are installed).</h4>
<p>The first time will take longer while image downloads.</p>
<pre><code>docker container run --rm -it -v $RC_DIR:/home/wruser/rc -v $(pwd):/home/wruser --workdir="/home/wruser" 
rmoorewrs/tic-sdk
</code></pre>
<h3 id="prepare-to-use-the-remote-clients">3 ) Prepare to use the remote clients</h3>
<p>Whether you’re running the tic-sdk docker container or using the TiC remote clients installed natively, you need to source the openrc credentials file for your user/project:</p>
<ul>
<li>substitute the name of your file for <code>my_openrc.sh</code></li>
</ul>
<pre><code>source rc/my_openrc.sh 
Please ... or press enter if you are not using HTTPS: &lt;enter&gt;
Please enter your OpenStack Password for project admin as user admin: &lt;password&gt; &lt;enter&gt;
</code></pre>
<h3 id="upload-the-cloud-image-file-with-glance-image-create-like-this">4 ) Upload the cloud image file with <code>glance image-create</code> like this:</h3>
<pre><code>glance image-create --visibility public --container-format bare --disk-format qcow2 --progress --file bionic-server-cloudimg-amd64.img --name bionic-server-cloudimg-amd64.img
</code></pre>
<p>When finished, you can check the availability of the image with:</p>
<pre><code>glance image-list
...
| 96eae3aa-9598-4b9c-9e3d-6eb5c3eb3e1a | bionic-server-cloudimg-amd64.img       |
...
</code></pre>
<h3 id="create-a-keypair-to-use-with-the-cloud-image-skip-if-you-have-an-existing-keypair-and-have-access-to-the-private-key">5) Create a keypair to use with the cloud image (skip if you have an existing keypair and have access to the private key)</h3>
<pre><code>openstack keypair create ubuntu_bionic_keypair
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA0KVB6w7b6Zf9821fbPoJ1sE6HYOWGaD0eJ30TqBiB2w2dzpL
+g9tFyz5/W/2RgqRGLxN266Am0RhW8BbkKutFce3KXE5HFaXmPv58FpKDW3yxRCS
...
KGG6jmEtQk25mD8tFnKXuocez1hIzgZTVuGF4wIEsqkqC5LSqnmC9nA23ZVTEYsF
7hhSQqclY9OV/bETTQKzmONhrLVo025Qpf5iicOGCMtHdageSk8D
-----END RSA PRIVATE KEY-----
</code></pre>
<h4 id="copy-and-paste-the-private-key-shown-on-the-screen-into-a-text-editor-and-save-including-the-lines-with-the----">5.1) Copy and paste the private key shown on the screen into a text editor and save (including the lines with the <code>---</code>)</h4>
<p><em><strong>NOTE: This is your only chance to save the private key!</strong></em><br>
Set the security of the private key (assume you called it <code>ubuntu_bionic_private.pem</code>)</p>
<pre><code>chmod 600 ubuntu_bionic_private.pem
</code></pre>
<h3 id="launch-the-instance-from-the-ubuntu-bionic-image-using--your-favorite-method-horizon-heat-nova-etc">6) Launch the Instance from the Ubuntu Bionic image using  your favorite method (horizon, heat, nova, etc)</h3>
<ul>
<li>make sure the instance is on a network that’s routable from your current machine (use floating IP if necessary)</li>
<li>find the IP address</li>
<li>NOTE: you won’t be able to log in via the console or by  ssh, without using the private key</li>
</ul>
<h4 id="connect-to-the-instance-using-ssh-and-the-private-key">6.1) Connect to the Instance using ssh and the private key</h4>
<pre><code>ssh -i ubuntu_bionic_private.pem ubuntu@&lt;instance IP addr&gt;
</code></pre>
<p>You should be able to log in with no other password</p>
<h3 id="configure-the-ubuntu-instance-to-allow-password-logins">7) Configure the Ubuntu Instance to allow password logins</h3>
<p>See full instructions <a href="https://fosskb.in/2015/02/06/enable-password-authentication-on-cloud-images/">here</a>:</p>
<pre><code>https://fosskb.in/2015/02/06/enable-password-authentication-on-cloud-images/
</code></pre>
<h4 id="edit-etcsshsshd_config-to-allow-password-authentication">7.1) Edit /etc/ssh/sshd_config to allow Password Authentication:</h4>
<pre><code>sudo vi /etc/ssh/sshd_config
</code></pre>
<p>Change <code>PasswordAuthentication</code> to yes:</p>
<pre><code>PasswordAuthentication yes
</code></pre>
<h4 id="set-a-password-for-ubuntu-andor-create-a-new-user">7.2) Set a password for <code>ubuntu</code> and/or create a new user</h4>
<p>By default the <code>ubuntu</code> user doesn’t have a password, so set it:</p>
<pre><code>sudo passwd ubuntu
</code></pre>
<p><strong>and/or</strong> create a new user (make sure and set the password):</p>
<pre><code>sudo useradd -m -s $(which bash) -G sudo mynewuser
sudo passwd mynewuser
</code></pre>
<h4 id="restart-sshd">7.2) Restart sshd</h4>
<pre><code>sudo systemctl restart sshd
</code></pre>
<h4 id="logout-and-log-back-in-with-new-user-or-ubuntu">7.3 Logout and Log back in with new user (or ubuntu)</h4>
<p>If all went well, you should now be able to log in without the private key</p>
<pre><code>ssh ubuntu@&lt;instance_ip_address&gt;
</code></pre>
<p>That’s it!</p>

