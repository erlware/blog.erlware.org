+++
author = "Tristan Sloughter"
type="post"
categories = ["Chef", "Devops", "Opscode"]
date = 2011-09-29T20:05:30Z
description = ""
draft = false
slug = "centos6-chef-node-creation"
tags = ["Chef", "Devops", "Opscode"]
title = "Centos6 : Chef Node Creation"

+++

I thought I'd share the scripts I use to take a fresh Centos6 install and have it configured to work with a Chef server. Maybe its not as easy as when running in a virtualized environment, but it saves plenty of time.  
  
On the new node I run the _setup_client.sh_ script which calls in the end _client_gen.sh_ on the Chef server once everything is installed on the node. I left the version numbers for Ruby and Chef in the script so you know what versions I've tested this with.  
  
## setup_client.sh  
  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#ff7f24;">#</span><span style="color:#ff7f24;">!/bin/</span><span style="color:#00ffff;">bash</span>  
<span style="color:#eedd82;">CHEF_IP</span>=XXX.XXX.XXX.XXX  
<span style="color:#eedd82;">CHEF</span>=http://$CHEF_IP:4000  
<span style="color:#eedd82;">CHEF_USER</span>=XXXXX  
<span style="color:#eedd82;">NODE</span>=XXXXXXXX  
<span style="color:#eedd82;">RUBY_VSN</span>=1.3.7  
<span style="color:#eedd82;">CHEF_VSN</span>=0.9.16  
  
sudo rpm -Uvh http://download.fedora.redhat.com/pub/epel/6/x86_64/epel-release-6-5.noarch.rpm  
  
sudo yum update  
  
sudo yum install ruby ruby-shadow ruby-ri ruby-rdoc gcc gcc-c++ ruby-devel ruby-static  
  
<span style="color:#b0c4de;">cd</span> /tmp  
wget http://production.cf.rubygems.org/rubygems/rubygems-$<span style="color:#eedd82;">RUBY_VSN</span>.tgz  
tar zxf rubygems-$<span style="color:#eedd82;">RUBY_VSN</span>.tgz  
<span style="color:#b0c4de;">cd</span> rubygems-$<span style="color:#eedd82;">RUBY_VSN</span>  
sudo ruby setup.rb --no-format-executable  
  
sudo gem install chef -v $<span style="color:#eedd82;">CHEF_VSN</span>   
  
mkdir ~/.chef  
  
cat &gt; ~/.chef/knife.rb &lt;&lt;EOF<span style="color:#ffff00;font-weight:bold;"> log_level :info log_location STDOUT node_name '$NODE' client_key '/home/$USER/.chef/$NODE.pem' validation_client_name 'chef-validator' validation_key '/etc/chef/validation.pem' chef_server_url '$CHEF' cache_type 'BasicFile' cache_options( :path =&gt; '/home/$USER/.chef/checksums' ) EOF</span>  
  
ssh-keygen -t rsa  
ssh-copy-id -i ~/.ssh/id_rsa.pub $<span style="color:#eedd82;">CHEF_IP</span>  
  
ssh $<span style="color:#eedd82;">CHEF_IP</span> <span style="color:#ffa07a;">"yes | knife client delete $NODE"</span>  
ssh $<span style="color:#eedd82;">CHEF_IP</span> <span style="color:#ffa07a;">"yes | /home/$CHEF_USER/client_gen.sh $NODE"</span>  
scp $<span style="color:#eedd82;">CHEF_IP</span>:/tmp/$<span style="color:#eedd82;">NODE</span> ~/.chef/$<span style="color:#eedd82;">NODE</span>.pem</pre>  
  
## client_gen.sh  
  
<pre style="color:#bebebe;background-color:#262626;"><span style="color:#ff7f24;">#</span><span style="color:#ff7f24;">!/bin/</span><span style="color:#00ffff;">bash</span>  
knife client create $<span style="color:#eedd82;">1</span> -n -a -f /tmp/$<span style="color:#eedd82;">1</span>  
knife node create $<span style="color:#eedd82;">1</span> --no-editor</pre>

