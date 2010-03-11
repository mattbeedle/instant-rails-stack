begin
  require 'vagrant'
rescue LoadError
  puts "The Vagrant gem is required in order to run these Rake tasks. Please install it first."
  puts "  [sudo] gem install vagrant"
  exit 1
end

task :default => [
  'vagrant:up',
  'vhosts:build',
  'vhosts:symlink',
  'apps:install_stack_requirements',
  'apps:install_gems',
  'apache:restart',
  'hosts:add'
]

namespace :vagrant do

  desc 'Boot up the Rails stack. Also available as \'vagrant up\''
  task :up do
    Vagrant::Commands.up
  end

end

namespace :vhosts do

  desc 'Build the VirtualHost configurations based on what applications are in apps/'
  task :build do
    each_app do |app_name|
      File.open("vhosts/#{app_name}", 'w') do |f|
        f.puts vhost_config_for(app_name)
      end
    end
    puts "** VirtualHost files built."
  end

  desc 'Symlink all VirtualHosts into /etc/apache2/sites-enabled on the guest OS'
  task :symlink do
    sudo("ln -sf #{vagrant_dir}/vhosts/* /etc/apache2/sites-enabled")
    puts "** VirtualHost files symlinked."
  end

  private

  def vhost_config_for(app_name)
    <<-EOS
RailsAutoDetect off
<VirtualHost *:80>
  ServerName #{app_name}
  RailsEnv development
  ErrorLog #{vagrant_dir}/logs/apache2/error.#{app_name}.log
  CustomLog #{vagrant_dir}/logs/apache2/access.#{app_name}.log combined
  DocumentRoot #{vagrant_dir}/apps/#{app_name}/public
  <Directory #{vagrant_dir}/apps/#{app_name}/public>
    Options FollowSymLinks
    Options -MultiViews
    AllowOverride All
    Order allow,deny
    Allow from all
  </Directory>
  RailsBaseURI /
</VirtualHost>
    EOS
  end

end

namespace :apps do

  desc 'Install the applications stack requirements'
  task :install_stack_requirements do
    puts "** Installing stack dependencies for all apps..."
    stack_requirements = Array.new
    each_app do |app_name|
      yaml_path = "apps/#{app_name}/stack_requirements.yml"
      next unless File.exist?(yaml_path)
      stack_requirements += YAML.load(IO.read(yaml_path))
    end
    execute("sudo apt-get -qyfm install #{stack_requirements.uniq.join(' ')}")
  end

  desc 'Install the applications gem dependencies'
  task :install_gems do
    each_app do |app_name|
      puts "** Installing gems for #{app_name}..."
      if File.exist?("apps/#{app_name}/Gemfile")
        install_bundler_if_needed
        sudo("cd #{vagrant_dir}/apps/#{app_name} && bundle install")
      else
        sudo("cd #{vagrant_dir}/apps/#{app_name} && rake gems:install")
      end
    end
    puts "** Application gems installed."
  end

  private

  def install_bundler_if_needed
    @bundler_installed ||= execute("sudo gem install bundler")
  end

end

namespace :apache do

  desc 'Restart Apache to make configuration changes effective.'
  task :restart do
    sudo("/etc/init.d/apache2 restart")
    puts "** Apache2 restarted successfully."
  end

end

namespace :hosts do

  desc 'Add Host aliases based on what applications are in apps/'
  task :add do
    load_ghost
    each_app do |app_name|
      Host.add(app_name, "127.0.0.1", true)
    end
    puts "** Application host aliases added."
  end

  private

  def load_ghost
    begin
      require 'ghost'
    rescue LoadError
      puts "Could not find the 'ghost' gem to add host aliases. Please install it."
      puts "  [sudo] gem install ghost"
      exit 1
    end
  end

end

private

def debug?
  ENV['DEBUG'].to_s == '1'
end

def each_app(&block)
  require 'pathname'
  Dir['apps/*'].each do |app_path|
    app_name = Pathname.new(app_path).basename.to_s
    yield(app_name)
  end
end

def vagrant_env_load!
  @vagrant_load_env ||= begin
    Vagrant::Env.load!
  end
end

def vagrant_config
  @vagrant_config ||= begin
    vagrant_env_load!
    Vagrant.config
  end
end

def vagrant_dir
  @vagrant_dir ||= vagrant_config.vm.project_directory
end

def rake_path
  @rake_path ||= execute('which rake').strip
end

def execute(cmd)
  vagrant_env_load!
  Vagrant::Env.require_persisted_vm

  result = nil
  Vagrant::SSH.execute do |ssh|
    puts cmd if debug?
    result = ssh.exec!(cmd)
    puts result if result
  end
  result
end

def sudo(cmd)
  execute("sudo env PATH=$PATH sh -c \"#{cmd}\"")
end
