# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Using a Trusty-based VM for building SecureDrop kernels using the `stable` patches.
  # Trusty boxes can't build `stable2` or `test` patches; see #30 for details:
  # https://github.com/freedomofpress/grsec/issues/30
  config.vm.define 'grsec-build-securedrop', primary: true do |build_sd|
    build_sd.vm.box = "bento/ubuntu-14.04"
    build_sd.vm.hostname = "grsec-build-securedrop"
    build_sd.vm.provision :ansible do |ansible|
      # Target the SecureDrop-specific playbook. Unfortunately Ansible won't
      # display the `vars_prompt` when run via vagrant, so you should actually
      # invoke `ansible-playbook` directly, like so:
      #
      # ansible-playbook -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory \
      # -u vagrant \
      # --private-key .vagrant/machines/grsec-build/libvirt/private_key \
      # examples/build-grsecurity-kernel-securedrop.yml
      #
      # Wish that weren't necessary, but it is.
      ansible.playbook = 'examples/build-grsecurity-kernel-securedrop.yml'
      ansible.verbose = 'vv'
    end
    build_sd.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.customize ["modifyvm", :id, "--cpus", available_vcpus]
    end
    build_sd.vm.provider "libvirt" do |v|
      v.memory = 2048
      v.cpus = available_vcpus
    end
  end

  # Deprecated machine intended to test grsecurity patches generally,
  # not specifically in the SecureDrop context. Vivid is EOL, so changes
  # are required to get this machine running again.
  config.vm.define 'grsec-build', autostart: false do |build|
    # Using Ubuntu 15.04 rather than 14.04 LTS due to a bug in kernel-package.
    # See #30 for details: https://github.com/freedomofpress/grsec/issues/30
    build.vm.box = "ubuntu/vivid64"
    build.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/vivid/current/vivid-server-cloudimg-amd64-vagrant-disk1.box"
    build.vm.hostname = "grsec-build"

    build.vm.provision :ansible do |ansible|
      # Two playbooks are available for building the kernel, for test and stable patches.
      # The stable playbook will prompt for HTTP auth credentials prior to running.
      # One subsequent runs of the stable playbook, you can simply hit enter to skip
      # the credential entry, since the files have already been downloaded.
      ansible.playbook = 'examples/build-grsecurity-kernel-stable.yml'
      ansible.playbook = 'examples/build-grsecurity-kernel-test.yml'
      ansible.verbose = 'vv'
      # Exposing the build strategy var for testing various build strategies quickly.
      ansible.extra_vars = {
        'grsecurity_build_strategy' => 'manual',
      }
    end
    build.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.customize ["modifyvm", :id, "--cpus", available_vcpus]
    end
    build.vm.provider "libvirt" do |v|
      v.memory = 2048
      v.cpus = available_vcpus
      # If you are experiencing CPU-compatibility warnings (esp. regarding lack
      # of svm) try uncommenting out the following line (see
      # https://github.com/freedomofpress/ansible-role-grsecurity/pull/85#issuecomment-266611369
      # for more).
      # v.cpu_mode = 'host-passthrough'
    end
  end

  # Separate machine for testing installation of .deb packages.
  # In case of problems, you don't want to pollute the build machine
  # with the test packages.
  config.vm.define 'grsec-install', autostart: false do |install|
    # Choose the base box for testing the grsecurity-patched kernel .deb package.
    # install.vm.box = "debian/wheezy64"
    # install.vm.box = "debian/jessie64"
    # install.vm.box = "ubuntu/vivid64"
    install.vm.box = "ubuntu/trusty64"
    install.vm.box_url = "https://cloud-images.ubuntu.com/vagrant/trusty/current/trusty-server-cloudimg-amd64-vagrant-disk1.box"
    install.vm.hostname = "grsec-install"
    # If grsec install works, the shared folder mount will fail.
    # Set `disabled: true` below to prevent the error post-install.
    install.vm.synced_folder './', '/vagrant', disabled: true
    install.vm.provider "virtualbox" do |v|
      v.gui = false
    end

    install.vm.provision :ansible do |ansible|
      ansible.playbook = 'examples/install-grsecurity-kernel.yml'
      ansible.extra_vars = {
        # Add the filename for the .deb package created by the build VM.
        # You may need to prefix the path with '../' if the .deb package is in the repo root.
        #'grsecurity_install_deb_package' => ''
      }
    end
  end
end


def available_vcpus
  # Increase number of virtual CPUs in guest VM.
  # Rather than blindly set it to "2" or similar,
  # inspect the number of VCPUs on the host and use that,
  # to minimize compile time.
  available_vcpus = case RUBY_PLATFORM
    when /linux/
      `nproc`.to_i
    when /darwin/
      `sysctl -n hw.ncpu`.to_i
    else
      1
    end
  # If you want to restrict the resources available to the guest VM,
  # uncomment the return line below, and Vagrant will use half the
  # number available on the host, rounded down.
  # (Ruby will correctly return the quotient as an integer.)
  # return available_vcpus / 2
  return available_vcpus
end

