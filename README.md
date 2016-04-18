# RaspberryPiを母艦に

RaspberryPiがどんなネットワークにいようと、raspiのIPアドレスを調べることなくmacbookからssh接続できるようにする。
これで、外出先でノマドしてても家にあるraspiの状態が見れるし、開発もできちゃうぞ

## 必要なもの

AWS/GCEにmicroやnanoインスタンス。月額300円ぐらい

## 簡単な説明

raspiからssh-tunnelインスタンスに常にssh Portforwardしている状態にします。いわゆるSSH Tunnelです。

### [macbook] <-> [(port:22)ssh-tunnel(port:20022)] <-> [raspi]


## 環境構築

### **raspi**
  - `ssh-keygen`
  - `cat ~/.ssh/id_rsa.pub`
    - ssh公開鍵をコピー

### **ssh-tunnel**

  - ssh-tunnelインスタンスを立ち上げ、タイプはmicroやnano
  - ssh接続し、**pi**ユーザー追加、ssh公開鍵を貼り付け
    - `ssh ssh-tunnel`
    - `useradd -s /bin/zsh -u 10000 -m -d /home/users/pi pi`
    - `sudo su - pi`
    - `mkdir .ssh`
    - `vim .ssh/authorized_keys`
      - ssh公開鍵を張り付け

### **macbook**
  - ~/.ssh/configに以下を追加。[]の部分は適宜書き換え

```
Host ssh-tunnel
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  User pi
  Hostname [SSH-TUNNEL IP]
  LogLevel QUIET
  IdentityFile ~/.ssh/google_compute_engine
```

  - 繋がるか確認
    - `ssh ssh-tunnel`

### **raspi**

  - ~/.ssh/configに以下を追加。[]の部分は適宜書き換え

```
Host ssh-tunnel
  StrictHostKeyChecking no
  UserKnownHostsFile /dev/null
  User [USERNAME]
  Hostname [SSH-TUNNEL IP]
  LogLevel QUIET
  IdentityFile ~/.ssh/google_compute_engine
```

  - 繋がるか確認
    - `ssh ssh-tunnel`
  -  起動スクリプトを書いて追加
    - `sudo apt-get install -y autossh`
    - `git clone https://github.com/mainyaa/RaspberryPi-Mothership.git`
    - `sudo cp ./RaspberryPi-Mothership/init.d/ssh_tunnel /etc/init.d/`
    - `sudo cp ./RaspberryPi-Mothership/sbin/ssh_tunnel /usr/sbin/`
    - `sudo insserv ssh_tunnel`
    - `sudo reboot`

### **macbook**（実際に繋いで見る）
  - 1度ssh-tunnelに接続。（ssh-tunnelがGCEの場合のみ)
      - `gcloud compute --project "[PROJECT_ID]" ssh --zone "asia-east1-b" "ssh-tunnel"`

## macbookからraspiに接続

### **`ssh -o 'ProxyCommand ssh ssh-tunnel nc %h %p' -p 20022 pi@localhost`**
  - これでRaspberryPiが母艦になりました！どこからでもssh-tunnelごしに繋がる!

## つっこみ大歓迎

間違ってたり、もっと便利な方法があったら教えて下さい！PullReqも歓迎です！

