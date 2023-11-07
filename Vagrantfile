# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

Vagrant.configure("2") do |config|
  ENV['VAGRANT_EXPERIMENTAL'] = "disks"
  ENV['SCENARIO'] ||= "aio"
  ENV['DEV_SRC_PATH'] ||= "/vagrant"
  ENV['BOOTSTRAP_OPTS'] ||= ""

  require "yaml"
  require "uri"

  config.vm.box = "ubuntu/jammy64"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = 12288
    vb.cpus = 4
  end

  config.vm.disk :disk, size: "100GB", primary: true

  a_r_r = YAML.load_file("ansible-role-requirements.yml")
  a_r_r.each do |requirement|
    src = URI(requirement["src"])
    src_name = src.path.sub(%r{^/}, "").split("/")[-1]

    if Dir.exist?("../#{src_name}")
      config.vm.synced_folder "../#{src_name}", "/vagrant/#{src.hostname}#{src.path}"
    end
  end

  a_c_r = YAML.load_file("ansible-collection-requirements.yml")
  a_c_r["collections"].each do |requirement|
    src = URI(requirement["source"])
    src_name = src.path.sub(%r{^/}, "").split("/")[-1]

    if Dir.exist?("../#{src_name}")
      config.vm.synced_folder "../#{src_name}", "/vagrant/#{src.hostname}#{src.path}"
    end
  end

  config.vm.synced_folder ".", "/opt/openstack-ansible"
  config.vm.synced_folder ".cache/apt", "/var/cache/apt", group: 'root', owner: 'root'
  config.vm.synced_folder ".cache/pip", "/root/.cache/pip", group: 'root', owner: 'root'
  config.vm.synced_folder ".cache/lxc", "/var/cache/lxc"
  config.vm.synced_folder ".repo", "/var/www/repo"

  config.vm.provision "shell", inline: <<-SHELL
    export SCENARIO='#{ENV["SCENARIO"]}'
    export DEV_SRC_PATH='#{ENV["DEV_SRC_PATH"]}'
    export BOOTSTRAP_OPTS='#{ENV["BOOTSTRAP_OPTS"]}'

    /opt/openstack-ansible/scripts/bootstrap-ansible.sh
    /opt/openstack-ansible/scripts/bootstrap-aio.sh

    test -f /opt/openstack-ansible/user_zzaio.yml && ln -sv /opt/openstack-ansible/user_zzaio.yml /etc/openstack_deploy/user_zzaio.yml

    cd /opt/openstack-ansible/playbooks

    openstack-ansible setup-everything.yml
  SHELL
end
