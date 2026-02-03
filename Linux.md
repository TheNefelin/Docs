## Ver erroes en Linux
* -b -1: Significa "mira el registro de la sesión anterior" (la que se bloqueó).
* -p 3: Filtra solo los errores graves (Errors). 
```sh
journalctl -b -1 -p 3
```

## Problema Asus X407UA se congela con Linux
* -b -1: Significa "mira el registro de la sesión anterior" (la que se bloqueó).
* -p 3: Filtra solo los errores graves (Errors).
```sh
journalctl -b -1 -p 3
```

## Instalar paquetes .xz
- decomprimir
```sh
ls
cd nombre-maquete
```

## Apps
### curl
```sh
sudo apt update && sudo apt install curl -y
```
### git
```sh
apt-get install git
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"   
```
### NodeJS 24
- Instala NVM
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc
nvm install --lts
node -v
npm -v
```
### Python 3.12
```sh
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt update
sudo apt install -y python3.12 python3.12-venv python3.12-dev python3-pip
```
- Nuevo projecto
- ./nuevo-projecto/app.py
```sh
python3.12 -m venv .venv
source .venv/bin/activate
python app.py
```
