# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/xenial64"

  config.vm.network "forwarded_port", guest: 3000, host: 3000
  config.vm.network "private_network", ip: "192.168.33.52"

  config.vm.synced_folder ".", "/vagrant"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "2048"
  end

  # https://github.com/lcreid/rails-5-jade
  config.vm.provision "shell", privileged: false, inline: <<-SHELL
    set -eux

    # Edit .bashrc
    sed -i -e 's/^HISTSIZE=1000$/HISTSIZE=100000/' ~/.bashrc
    sed -i -e 's/^HISTFILESIZE=2000$/HISTFILESIZE=200000/' ~/.bashrc
    if ! grep -q 'PROMPT_COMMAND' ~/.bashrc; then
      echo "\n"'PROMPT_COMMAND="history -a; history -n"' >> ~/.bashrc
    fi

    # Allocate swap
    if ! grep -q 'swapfile' /etc/fstab; then
      SWAPSIZE=4G

      sudo fallocate -l ${SWAPSIZE} /swapfile
      sudo chmod 600 /swapfile
      sudo mkswap /swapfile
      sudo swapon /swapfile
      echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab
    fi

    # To fix warning: 'dpkg-reconfigure: unable to re-open stdin: No file or directory'
    sudo ex +"%s@^DPkg@//DPkg" -cwq /etc/apt/apt.conf.d/70debconf \
      && sudo dpkg-reconfigure debconf -f noninteractive -p critical

    # Set timezone
    sudo timedatectl set-timezone Asia/Tokyo

    # apt-get update
    sudo apt-get update

    # Install essentials
    sudo apt-get install -y make gcc

    # Install git
    sudo apt-get install -y git

    # Install rbenv & ruby-build
    if ! [ -e $HOME/.rbenv/bin/rbenv ]; then
      sudo apt-get install -y libreadline-dev
      git clone https://github.com/rbenv/rbenv.git ~/.rbenv
      cd ~/.rbenv && src/configure && make -C src
      cat << 'EOM' >> ~/.bashrc

export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
EOM
      source ~/.bashrc

      git clone https://github.com/rbenv/ruby-build.git
      $(cd ruby-build && sudo ./install.sh)
    fi

    # Build default Ruby
    sudo apt-get install -y libssl-dev zlib1g-dev
    $HOME/.rbenv/bin/rbenv rehash
    $HOME/.rbenv/bin/rbenv install --skip-existing 2.5.1
    $HOME/.rbenv/bin/rbenv global 2.5.1
    $HOME/.rbenv/bin/rbenv rehash

    # Nokogiri build dependencies (from http://www.nokogiri.org/tutorials/installing_nokogiri.html#ubuntu___debian)
    sudo apt-get install -y patch zlib1g-dev liblzma-dev

    # Install nodenv & node-build
    if ! [ -e $HOME/.nodenv/bin/nodenv ]; then
      sudo apt-get install -y libreadline-dev
      git clone https://github.com/nodenv/nodenv.git ~/.nodenv
      cd ~/.nodenv && src/configure && make -C src
      cat << 'EOM' >> ~/.bashrc

export PATH="$HOME/.nodenv/bin:$PATH"
eval "$(nodenv init -)"
EOM
      source ~/.bashrc

      git clone https://github.com/nodenv/node-build.git
      $(cd node-build && sudo ./install.sh)
    fi

    # Build default Node
    $HOME/.nodenv/bin/nodenv rehash
    $HOME/.nodenv/bin/nodenv install --skip-existing 8.11.1
    $HOME/.nodenv/bin/nodenv global 8.11.1
    $HOME/.nodenv/bin/nodenv rehash

    # Install Yarn
    which yarn || {
      curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
      echo "deb https://dl.yarnpkg.com/debian/ stable main" |
        sudo tee /etc/apt/sources.list.d/yarn.list
      sudo apt-get update
      sudo apt-get install -y yarn
    }

    # Install Postgres
    apt-cache show postgresql-10 || {
      sudo add-apt-repository 'deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main'
      wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
      sudo apt-get update
      sudo apt-get install -y postgresql-10 libpq-dev
      sudo -u postgres psql -c "CREATE ROLE vagrant WITH CREATEDB LOGIN;"
    }

    # Install Redis
    # Adapted from: https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-redis-on-ubuntu-16-04
    if ! [ -e /etc/systemd/system/redis.service ]; then
      sudo apt-get install -y tcl
      curl -O http://download.redis.io/redis-stable.tar.gz
      tar -xzvf redis-stable.tar.gz
      cd redis-stable
      make
      sudo make install
      sudo adduser --system --group --no-create-home redis
      sudo mkdir /var/lib/redis /var/log/redis
      sudo chown redis:redis /var/lib/redis /var/log/redis
      sudo chmod 770 /var/lib/redis /var/log/redis
      sudo sed -i.original \
        -e '/^supervised no/s/no/systemd/' \
        -e '/^dir/s;.*;dir /var/lib/redis;' \
        redis.conf
      sudo cp redis.conf /etc
      cat >redis.service <<EOF
[Unit]
Description=Redis In-Memory Data Store
After=network.target
[Service]
User=redis
Group=redis
ExecStart=/usr/local/bin/redis-server /etc/redis/redis.conf
ExecStop=/usr/local/bin/redis-cli shutdown
Restart=always
[Install]
WantedBy=multi-user.target
EOF
      sudo cp redis.service /etc/systemd/system/
      cd ..
    fi

    # Sendmail
    sudo apt-get install -y sendmail

    # Install Chrome because the world is moving to headless Chrome.
    # From: https://askubuntu.com/a/510186/264753
    which google-chrome-stable || {
      wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add
      echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list
      sudo apt-get -y update
      sudo apt-get -y install google-chrome-stable
    }

    # Need the following if you're going to build webkit for Capybara
    sudo apt-get install -y libqtwebkit-dev gstreamer1.0-plugins-base gstreamer1.0-tools gstreamer1.0-x xvfb

    # Install Heroku Toolbelt
    which heroku || {
      wget -qO- https://toolbelt.heroku.com/install.sh | sh
      echo "\n"'PATH="/usr/local/heroku/bin:$PATH"' >> ~/.bashrc
    }

    # cleanup apt & upgrade
    sudo apt-get update
    sudo apt-get -y dist-upgrade
    sudo apt-get -y autoremove

    # Install peco
    which peco || {
      mkdir ~/bin
      curl -L https://github.com/peco/peco/releases/download/v0.5.3/peco_linux_amd64.tar.gz | tar -xzC /tmp
      mv /tmp/peco_linux_amd64/peco ~/bin
      cat << 'EOM' >> ~/.bashrc

function peco-select-history() {
    local tac
    which gtac &> /dev/null && tac="gtac" || \\
        which tac &> /dev/null && tac="tac" || \\
        tac="tail -r"
    READLINE_LINE=$(HISTTIMEFORMAT= history | $tac | sed -e 's/^\\s*[0-9]\\+\\s\\+//' | awk '!a[$0]++' | peco --query "$READLINE_LINE")
    READLINE_POINT=${#READLINE_LINE}
}
bind -x '"\\C-r": peco-select-history'
EOM
    }

    # Install gems
    $HOME/.rbenv/shims/gem install bundler
  SHELL
end
