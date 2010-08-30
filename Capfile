#
# Chef-Solo Capistrano Bootstrap
#
# usage:
#   cap chef:bootstrap <role> <remote_host>
#
# NOTICE OF LICENSE
#
# Copyright (c) 2010 Mike Smullin <mike@smullindesign.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# configuration
set :port, 22
default_run_options[:pty] = true # fix to display interactive password prompts
role :target, ARGV[-1]
cwd = File.expand_path(File.dirname(__FILE__))
cookbook_dir = '/var/chef-solo'
dna_dir = '/etc/chef'
node = ARGV[-2]

# tasks
namespace :chef do
  desc "Bootstrap an Ubuntu 10.04 server and kick-start Chef-Solo"
  task :bootstrap, roles: :target do
    install_rvm_ruby
    #install_ree
    install_chef
    install_cookbook_repo
    install_dna
    solo
  end

  desc "Install Ruby Enterprise Edition"
  # see also: http://www.rubyenterpriseedition.com/download.html
  task :install_ree, roles: :target do
    ree_version = 'ruby-enterprise-1.8.7-2010.02'
    ree_prefix = '/usr/local'
    sudo 'aptitude install -y zlib1g-dev libssl-dev libreadline5-dev' # REE dependencies
    run "cd /tmp && wget http://rubyforge.org/frs/download.php/71096/#{ree_version}.tar.gz && tar zxvf #{ree_version}.tar.gz"
    sudo "mkdir -p #{ree_prefix}/lib/ruby/gems/1.8/gems" # workaround for bug in REE installer
    sudo "/tmp/#{ree_version}/installer -a #{ree_prefix} --dont-install-useful-gems --no-dev-docs"
    run "echo 'gem: --no-ri --no-rdoc' | #{sudo} tee -a /etc/gemrc"
  end

  desc "install Ruby (using RVM)"
  # see also: http://rohitarondekar.com/articles/installing-rails3-beta3-on-ubuntu-using-rvm
  task :install_rvm_ruby, roles: :target do
    rvm_ruby_version = '1.9.2-p0'
    set :default_environment, {
      'PATH'         => "/usr/local/rvm/gems/ruby-#{rvm_ruby_version}/bin:/usr/local/rvm/gems/ruby-#{rvm_ruby_version}@global/bin:/usr/local/rvm/rubies/ruby-#{rvm_ruby_version}/bin:$PATH",
      'RUBY_VERSION' => "ruby #{rvm_ruby_version}",
      'GEM_HOME'     => "/usr/local/rvm/gems/ruby-#{rvm_ruby_version}",
      'GEM_PATH'     => "/usr/local/rvm/gems/ruby-#{rvm_ruby_version}:/usr/local/rvm/gems/ruby-#{rvm_ruby_version}@global",
      'BUNDLE_PATH'  => "/usr/local/rvm/gems/ruby-#{rvm_ruby_version}"
    }
    msudo [
      # install RVM
      'aptitude install -y curl git-core',
      "curl -L http://bit.ly/rvm-install-system-wide | #{sudo} bash",
      %q(sed -i 's/^\[/# [/' /root/.bashrc),
      %q(echo '[[ -s "/usr/local/rvm/scripts/rvm" ]] && source "/usr/local/rvm/scripts/rvm"' | sudo tee -a /root/.bashrc),

      # dependencies for compiling Ruby
      'aptitude install -y bison build-essential autoconf zlib1g-dev libssl-dev libxml2-dev libreadline6-dev',

      # RVM packages for Ruby
      'rvm package install openssl',
      'rvm package install readline',

      # install Ruby
      "rvm install #{rvm_ruby_version} -C --with-openssl-dir=$rvm_path/usr,--with-readline-dir=$rvm_path/usr",
      "rvm use #{rvm_ruby_version} --default",
      "echo 'gem: --no-ri --no-rdoc' | #{sudo} tee -a /etc/gemrc" # saves time; don't need docs on server
    ]
    mrun [
      "#{sudo} usermod -a -G rvm `whoami`", # add user to the RVM group
      %q(sed -i 's/^\[/# [/' ~/.bashrc),
      %q(echo '[[ -s "/usr/local/rvm/scripts/rvm" ]] && source "/usr/local/rvm/scripts/rvm"' | tee -a ~/.bashrc)
    ]
  end

  desc "Install Chef and Ohai gems as root"
  task :install_chef, roles: :target do
    sudo 'gem source -a http://gems.opscode.com/'
    sudo 'gem install ohai chef'
  end

  desc "Install Cookbook Repository from cwd"
  task :install_cookbook_repo, roles: :target do
    sudo 'aptitude install -y rsync'
    sudo "mkdir -m 0775 -p #{cookbook_dir}"
    sudo "chown `whoami`.`whoami` #{cookbook_dir}"
    reinstall_cookbook_repo
  end

  desc "Re-install Cookbook Repository from cwd"
  task :reinstall_cookbook_repo, roles: :target do
    rsync cwd + '/', cookbook_dir
  end

  desc "Install ./dna/*.json for specified node"
  task :install_dna, roles: :target do
    sudo 'aptitude install -y rsync'
    sudo "mkdir -m 0775 -p #{dna_dir}"
    sudo "chown `whoami`.`whoami` #{dna_dir}"
    put %Q(file_cache_path "#{cookbook_dir}"
cookbook_path ["#{cookbook_dir}/cookbooks", "#{cookbook_dir}/site-cookbooks"]
role_path "#{cookbook_dir}/roles"), "#{dna_dir}/solo.rb", via: :scp, mode: "0644"
    reinstall_dna
  end

  desc "Re-install ./dna/*.json for specified node"
  task :reinstall_dna, roles: :target do
    rsync "#{cwd}/dna/#{node}.json", "#{dna_dir}/dna.json"
  end

  desc "Execute Chef-Solo"
  task :solo, roles: :target do
    sudo_env "chef-solo -c #{dna_dir}/solo.rb -j #{dna_dir}/dna.json -l debug"

    exit # subsequent args are not tasks to be run
  end

  desc "Remove all traces of Chef"
  task :cleanup, roles: :target do
    msudo [
      "rm -rf #{dna_dir} #{cookbook_dir}",
      'gem uninstall -ay chef ohai'
    ]
  end
end


# helpers
def sudo_env(cmd)
  run "#{sudo} -i #{cmd}"
end

def msudo(cmds)
  cmds.each do |cmd|
    sudo cmd
  end
end

def mrun(cmds)
  cmds.each do |cmd|
    run cmd
  end
end

def rsync(from, to)
  find_servers_for_task(current_task).each do |server|
    puts `rsync -avz -e "ssh -p#{port}" "#{from}" "#{ENV['USER']}@#{server}:#{to}" \
      --exclude ".svn" --exclude ".git"`
  end
end
