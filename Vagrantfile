# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|

    # Name the box which will be worked on
    config.vm.box = "debSqueeze64"

    # Update the hostname to something descriptive and personalised
    config.vm.hostname = "debsqueeze4globalx.example.com"

    config.vm.provider "virtualbox" do |vb|

        # Ensure that the GUI can be seen
        vb.gui = true

        # Customise memory usage. Keep it low, more isn't needed.
        vb.memory = "512"

        # Update the network to resolve everything as I don't know the setup
        # at GlobalX. This should allow internet access from the created server.
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]

    end

    config.vm.post_up_message = "Steve Nicklin's Debian Squeeze Example Box for GlobalX"

    config.vm.provision "shell", inline: <<-SHELL

        # Download a Puppet config file to install emacs
        # It is a .html file as the webserver insists on .html being served
        echo "Downloading emacs manifest file"
        wget http://leapforwards.co.uk/emacs.html -P /etc/puppet/manifests
        if [ $? -ne 0 ]
        then
            echo "Cannot download emacs.html from www. Exiting..."
            exit 1
        fi

        mv /etc/puppet/manifests/emacs.html /etc/puppet/manifests/emacs.pp
        if [ $? -ne 0 ]
        then
            echo "Cannot move emacs.html. Exiting..."
            exit 1
        fi

        # Set some values for Puppet information
        PUPPETFILE=/etc/default/puppet
        SAVEFILE=~/puppet

        # Copy the file to the home directory for safe keeping 
        echo "Saving puppet file to home directory..."
        cp -p ${PUPPETFILE} ${SAVEFILE} 
        if [ $? -ne 0 ]
        then
            echo "Cannot copy ${PUPPETFILE} to home directory, exiting..."
            exit 1
        fi

        # Update the puppet file to start on boot
        echo "Updating the puppet file to start on next boot"

        sed 's/START\=no/START\=yes/g' ${PUPPETFILE} > ${PUPPETFILE}.tmp
        if [ $? -ne 0 ]
        then
            echo "Could not update ${PUPPETFILE}.tmp, exiting..."
            exit 1
        fi
        mv ${PUPPETFILE}.tmp ${PUPPETFILE}
        if [ $? -ne 0 ]
        then
            echo "Could not update ${PUPPETFILE}, exiting..."
            exit 1
        fi

        # Start puppet up for the first time
        echo "Starting puppet"
        service puppet start
        if [ $? -ne 0 ] 
        then
            echo "Could not start puppet, please check logs, exiting..."
            exit 1
        fi

        # Lets load MongoDB onto the OS, it's a cool DB.
        echo "Updating APT"
        sudo apt-get update
        if [ $? -ne 0 ]
        then
            echo "Could not update apt, exiting..."
            exit 1
        fi

        echo "Installing mongodb using apt-get"
        sudo apt-get install -y mongodb
        if [ $? -ne 0 ]
        then
            echo "Could not install mongodb, exiting..."
            exit 1
        fi
     
        # Completing an initial run of puppet so that we don't have to wait
        # As well as this initial run, there will be a file in place in the
        # manifests directory, so if someone removes emacs, it should be
        # reinstalled back where it was
        echo "Checking if emacs is installed, on initial run it shouldn't be." 
        echo "If it isn't there, it'll be installed."
        puppet apply /etc/puppet/manifests/emacs.pp
        if [ $? -ne 0 ]
        then
            echo "Could not apply emacs.pp"
            exit 1
        fi
  
  SHELL
end
