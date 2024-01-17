+++
title = 'ตั้งค่า Ubuntu Server'
date = 2024-01-14T07:07:07+01:00
+++

## Introduction

โน๊ตนี้คือ commands ที่มักจะใช้บ่อย ๆ เมื่อต้อง setup ubuntu server (20.04, 22.04)

-   เลือกติดตั้งตามที่ต้องใช้นะ

#### 1. Basic setup

```sh
sudo apt update && sudo apt upgrade -y
sudo timedatectl set-timezone Asia/Bangkok
sudo apt install build-essential
```

---

#### 2. Install docker

```sh
sudo apt update
sudo apt install curl -y

cd /tmp
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# run docker without sudo
sudo gpasswd -a $USER docker
sudo service docker restart
```

##### ทดสอบ

```
docker --version
```

---

#### 3. Install ZSH + P10k Configure

```sh
sudo apt install curl wget git zsh -y

sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Install plugins
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Install fonts
sudo apt install fonts-powerline
```

##### Update `.zshrc` file

```sh
nano ~/.zshrc

# Update plugins
plugins=(
    git
    zsh-syntax-highlighting # <- Add
    zsh-autosuggestions     # <- Add
)
```

```sh
source ~/.zshrc
```

##### Setup P10K Configure

```sh
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# Update .zshrc file
nano ~/.zshrc

# Update Theme
ZSH_THEME="powerlevel10k/powerlevel10k"

# Reload
source ~/.zshrc
```

---

#### 4. Setup NodeJS (v.20)

```sh
curl -sL https://deb.nodesource.com/setup_20.x -o /tmp/nodesource_setup.sh

sudo bash /tmp/nodesource_setup.sh

sudo apt install nodejs

# Verify
node -v

# Install yarn
sudo npm install -g yarn

# Verify
yarn --version
```
