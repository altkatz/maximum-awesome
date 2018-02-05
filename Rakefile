def brew_install(package, *args)
  versions = `brew list #{package} --versions`
  options = args.last.is_a?(Hash) ? args.pop : {}

  # if brew exits with error we install tmux
  if versions.empty?
    sh "brew install #{package} #{args.join ' '}"
  elsif options[:requires]
    # brew did not error out, verify tmux is greater than 1.8
    # e.g. brew_tmux_query = 'tmux 1.9a'
    installed_version = versions.split(/\n/).first.split(' ').last
    unless version_match?(options[:requires], installed_version)
      sh "brew upgrade #{package} #{args.join ' '}"
    end
  end
end

def version_match?(requirement, version)
  # This is a hack, but it lets us avoid a gem dep for version checking.
  # Gem dependencies must be numeric, so we remove non-numeric characters here.
  matches = version.match(/(?<versionish>\d+\.\d+)/)
  return false unless matches.length > 0
  Gem::Dependency.new('', requirement).match?('', matches.captures[0])
end

def install_github_bundle(user, package)
  unless File.exist? File.expand_path("~/.vim/bundle/#{package}")
    sh "git clone https://github.com/#{user}/#{package} ~/.vim/bundle/#{package}"
  end
end

def step(description)
  description = "-- #{description} "
  description = description.ljust(80, '-')
  puts
  puts "\e[32m#{description}\e[0m"
end


def get_backup_path(path)
  number = 1
  backup_path = "#{path}.bak"
  loop do
    if number > 1
      backup_path = "#{backup_path}#{number}"
    end
    if File.exists?(backup_path) || File.symlink?(backup_path)
      number += 1
      next
    end
    break
  end
  backup_path
end

def link_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.exists?(symlink_path) || File.symlink?(symlink_path)
    if File.symlink?(symlink_path)
      symlink_points_to_path = File.readlink(symlink_path)
      return if symlink_points_to_path == original_path
      # Symlinks can't just be moved like regular files. Recreate old one, and
      # remove it.
      ln_s symlink_points_to_path, get_backup_path(symlink_path), :verbose => true
      rm symlink_path
    else
      # Never move user's files without creating backups first
      mv symlink_path, get_backup_path(symlink_path), :verbose => true
    end
  end
  ln_s original_path, symlink_path, :verbose => true
end

def unlink_file(original_filename, symlink_filename)
  original_path = File.expand_path(original_filename)
  symlink_path = File.expand_path(symlink_filename)
  if File.symlink?(symlink_path)
    symlink_points_to_path = File.readlink(symlink_path)
    if symlink_points_to_path == original_path
      # the symlink is installed, so we should uninstall it
      rm_f symlink_path, :verbose => true
      backups = Dir["#{symlink_path}*.bak"]
      case backups.size
      when 0
        # nothing to do
      when 1
        mv backups.first, symlink_path, :verbose => true
      else
        $stderr.puts "found #{backups.size} backups for #{symlink_path}, please restore the one you want."
      end
    else
      $stderr.puts "#{symlink_path} does not point to #{original_path}, skipping."
    end
  else
    $stderr.puts "#{symlink_path} is not a symlink, skipping."
  end
end

def filemap(map)
  map.inject({}) do |result, (key, value)|
    result[File.expand_path(key)] = File.expand_path(value)
    result
  end.freeze
end

DIRS_UNDER_HOME = [
  '~/.config/nvim/',
  '~/.local/share/nvim/site',
]
LINKED_FILES = filemap(
  'vim'           => '~/.vim',
  'tmux.conf'     => '~/.tmux.conf',
  'vimrc'         => '~/.vimrc',
  'vimrc.plugs' => '~/.vimrc.plugs',
  'vim/autoload' => '~/.local/share/nvim/site/autoload',
  '~/.vimrc'       => '~/.config/nvim/init.vim',
)

#ln -s ~/.vim/autoload ~/.local/share/nvim/site
desc 'Install these config files.'
task :install do
  # TODO install gem ctags?
  # TODO run gem ctags?
  DIRS_UNDER_HOME.each do |dir|
    FileUtils.mkdir_p(File.expand_path(dir))
  end
  step 'symlink'
  LINKED_FILES.each do |orig, link|
    link_file orig, link
  end
end

desc 'Uninstall these config files.'
task :uninstall do
  step 'un-symlink'
  # un-symlink files that still point to the installed locations
  LINKED_FILES.each do |orig, link|
    unlink_file orig, link
  end
end

task :default => :install
