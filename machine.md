# OS install parameter
コード化も可能だが後手に回したので、設定手順のメモを設置する  
  
## OSインストール時のパラメーター
◆VMスペック  
cpu 1core , mem 2G , hdd 16G , 管理用Networkを付与したNIC  
  
◆仮想マシンの作成  
名前の指定  
クラスタ、リソース、ストレージの指定  
linux , ubuntu20 64bit  
リソースの指定でコンテンツライブラリのisoを指定して接続にチェック  
  
起動  
言語  
　Englishを選択  
キーボード選択  
　LayoutでJapaneseを選択  
Network  
　Continue without networkを選択  
Proxy,Mirror,Updatecheck  
　操作せずDone  
diskとfilesystem  
　指定のディスク容量でLVMが設定されている  
　そのままDoneを選択  
Profile  
　osmanager, ubuntu-20.x-masterimg,  
advantage  
　Done  
InstallOpenSSHserver check  
　Done  
Reboot  
Enter  
  
  
## セットアップコマンド
◆OS設定  
osmanagerでログイン  
sudo でrootとなり設定する  
  
IP設定  
vim /etc/netplan/50-config.yaml
```
network:
    ethernets:
        ens0:
            optional: true
            dhcp4: false
            addresses:
            - 192.168.10.245/24
            gateway4: 192.168.10.1
            nameservers:
                addresses:
                - 192.168.10.1
    version: 2
```
  
システム設定
```
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.org
cat /etc/ssh/sshd_config | sed 35i'PermitRootLogin no \n' > /etc/ssh/sshd_config
echo -e "\nnet.ipv6.conf.all.disable_ipv6=1 \nnet.ipv6.conf.default.disable_ipv6=1 \nnet.ipv6.conf.lo.disable_ipv6=1 \n" >> /etc/sysctl.conf

cp /usr/share/zoneinfo/Asia/Tokyo /etc/localtime 
echo -e "\nNTP=192.168.10.1\n" >> /etc/systemd/timesyncd.conf
```
package導入
```
apt update
apt install open-vm-tools
apt install net-tools
```
OSの停止
```
rm -f /etc/netplan/50-config.yaml
shutdown -h now
```
  
vCenterで設定
```
設定の編集でCD/DVDをクライアントデバイスに変える
annotationの付与 -> 20220519 created  
VMのtemplate化  
playbookで作成されるnic構成ファイルは99-vmware-configured.ymlとなる  
```
  