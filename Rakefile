require 'rake'
require 'erb'

desc "install the dot files into user's home directory"
task :install do
  replace_all = false

  install_utilities

  Dir['*'].each do |file|
    next if %w[Rakefile Readme.md LICENSE].include? file
    
    if File.exist?(File.join(ENV['HOME'], ".#{file.sub('.erb', '')}"))
      if File.identical? file, File.join(ENV['HOME'], ".#{file.sub('.erb', '')}")
        puts "identical ~/.#{file.sub('.erb', '')}"
      elsif replace_all
        replace_file(file)
      else
        print "overwrite ~/.#{file.sub('.erb', '')}? [ynaq] "
        case $stdin.gets.chomp
        when 'a'
          replace_all = true
          replace_file(file)
        when 'y'
          replace_file(file)
        when 'q'
          exit
        else
          puts "skipping ~/.#{file.sub('.erb', '')}"
        end
      end
    else
      link_file(file)
    end
  end
end

def install_utilities
  install_homebrew
  install_homebrew_utils
  install_ruby("3.4.6")
  install_node("0.40.3")
  install_npm_packages
  install_gems
  install_dotnet
  install_flutter("3.35.4")
  install_terraform()
  install_omz
end

def install_homebrew
  puts "Installing homebrew"

  if command_found?("brew")
    system('brew update && brew upgrade', out: STDOUT)
  else
    system('/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"', out: STDOUT) unless command_found?("brew")
  end
end

def install_homebrew_utils
  puts "Installing rbenv"
  system('brew install rbenv', out: STDOUT) unless command_found?("rbenv")

  puts "Installing thefuck"
  system('brew install thefuck', out: STDOUT) unless command_found?("thefuck")

  puts "Installing zsh-syntax-highlighting"
  system('brew install zsh-syntax-highlighting', out: STDOUT) unless File.exist?("/usr/local/share/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh")

  puts "Installing zsh-history-substring-search"
  system('brew install zsh-history-substring-search', out: STDOUT) unless File.exist?("/usr/local/share/zsh-history-substring-search/zsh-history-substring-search.zsh")

  puts "Installing starship"
  system('brew install starship') unless command_found?("starship")

  puts "Installing Golang"
  system('brew install go') unless command_found?("go")

  puts "Installing azure-cli"
  system('brew install azure-cli') unless command_found?("az")

  puts "Installing gh-cli"
  system('brew install gh') unless command_found?("gh")

  puts "Installing openssl@3"
  system('brew install openssl@3', out: STDOUT) unless command_found?("openssl")

  puts "Installing firebase cli"
  system('brew install firebase-cli', out: STDOUT) unless command_found?("firebase")

  puts "Installing tfenv"
  system('brew install tfenv', out: STDOUT) unless command_found?("tfenv")

  puts "Installing Nerdfonts"
  system('brew install --cask font-fira-code-nerd-font', out:STDOUT)

  puts "Installing OpenJDK"
  system('brew install openjdk')
end

def install_ruby(ruby_version)
  openssl_prefix = `brew --prefix openssl@3`.strip

  env = {
    "LDFLAGS" => "-L#{openssl_prefix}/lib",
    "CPPFLAGS" => "-I#{openssl_prefix}/include",
    "PKG_CONFIG_PATH" => "#{openssl_prefix}/lib/pkgconfig",
    "RUBY_CONFIGURE_OPTS" => "--with-openssl-dir=#{openssl_prefix}"
  }
  
  puts "Installing Ruby #{ruby_version}"
  success = system(env, "rbenv install -f #{ruby_version}", out: STDOUT)
  if success
    system("rbenv global #{ruby_version}")
    system("rbenv rehash")
    puts "Ruby #{ruby_version} installed and set as global version."
  else
    abort "Ruby installation failed."
  end
end

def install_terraform(version="latest")
  puts "Installing #{version} Terraform"
  success = system("tfenv install #{version}", out: STDOUT)
  if success
    system("tfenv use #{version}")
  else
    abort "Terraform installation failed."
  end
end

def install_flutter(version)
  flutter_tar = "flutter_macos_arm64_#{version}-stable.zip"

  puts "Installing flutter"
  system("curl -o #{flutter_tar} https://storage.googleapis.com/flutter_infra_release/releases/stable/macos/#{flutter_tar}", out: STDOUT) unless command_found?("flutter")

  system("unzip #{flutter_tar} -d ~/development/", out: STDOUT) unless command_found?("flutter")
end

def install_node(version)
  puts "Installing nvm"
  system("curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v#{version}/install.sh | bash", out: STDOUT) unless File.exist?("/Users/dwhathaway/.nvm/nvm.sh")

  puts "Installing node"
  system('nvm install node', out: STDOUT) unless command_found?("node")
end

def install_npm_packages
  puts "No packages to install.."
end

def install_gems
  puts "Installing colorls"
  system('gem install colorls', out: STDOUT) unless command_found?("colorls")

  puts "Installing cocoapods"
  system('gem install cocoapods', out: STDOUT) unless command_found?("cocoapods")
end

def install_dotnet
  puts "Installing dotnet sdk"
  # installs LTS by default
  system('curl -sSL https://dot.net/v1/dotnet-install.sh | bash /dev/stdin', out: STDOUT) unless command_found?("dotnet")
end

def install_omz
  puts "Installing oh my zsh"
  system('sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"')
end

def command_found?(command)
  system("which #{command} > /dev/null 2>&1")
end

def replace_file(file)
  system %Q{rm -rf "$HOME/.#{file.sub('.erb', '')}"}
  link_file(file)
end

def link_file(file)
  if file =~ /.erb$/
    puts "generating ~/.#{file.sub('.erb', '')}"
    File.open(File.join(ENV['HOME'], ".#{file.sub('.erb', '')}"), 'w') do |new_file|
      new_file.write ERB.new(File.read(file)).result(binding)
    end
  else
    puts "linking ~/.#{file}"
    system %Q{ln -s "$PWD/#{file}" "$HOME/.#{file}"}
  end
end
