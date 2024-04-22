# python
```
apt-get install wget
wget https://bootstrap.pypa.io/pip/pip.pyz
sudo apt install python3.7-distutils
python -m pip.pyz --help
```
# .net core
* download 3.1 sdk
  https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-3.1.426-linux-arm32-binaries
```
  mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-3.1.426-linux-arm.tar.gz -C $HOME/dotnet
  export DOTNET_ROOT=$HOME/dotnet
  export PATH=$PATH:$HOME/dotnet
```
```
root@ebox:/home/yunzao/publish# nano ~/.bashrc
```
paste that 2 line into the most below:
```
export DOTNET_ROOT=$HOME/dotnet
export PATH=$PATH:$HOME/dotnet

```
disable the globalization:
```
export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1
```
