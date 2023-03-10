# Cara deploy pingpub expoler + reverse proxy nginx
## Yang di butuhkan
- **Domain explorer dan vps webserver untuk hosting explorernya**
- **Subdoaamin rpc dan api nya**
- **Nginx buat reverse proxy**
- **Repo ping pub buat deploy explorernya**

## Di node API nya jadiin true
- **Kalo udah di truein restart nodenya**
## Install paket yang di butuhkan
```
sudo apt update && sudo apt upgrade -y
sudo apt install nginx certbot python3-certbot-nginx -y
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get update && apt install -y nodejs git
curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get install yarn -y

```
## reverse proxy nginx + install ssl
##### 1. Config API
```
nano /etc/nginx/sites-enabled/<YOUR.API.SUBDOMAIN.SITE>.conf
```
##### 2. Copy seperti di bawah ini
```
server {
    server_name YOUR.API.SUBDOMAIN.SITE;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://YOUR_API_NODE_IP:1337;

    }
}

```
- ** `YOUR.API.SUBDOMAIN.SITE` ganti dengan subdomain api anda `http://YOUR_API_NODE_IP:1337` ganti dengan ip node dan port apinya sesuaikan yg di node**
##### 3. Config RPC
```
nano /etc/nginx/sites-enabled/<YOUR.RPC.SUBDOMAIN.SITE>.conf
```
##### 4. Copy seperti di bawah ini:
```
server {
    server_name YOUR.RPC.SUBDOMAIN.SITE;
    listen 80;
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;

	proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header   Host             $host;

        proxy_pass http://YOUR_RPC_NODE_IP:26657;

    }
}

```
- ** `YOUR.RPC.SUBDOMAIN.SITE` ganti dengan subdomain rpc anda, `http://YOUR_API_NODE_IP:26657` ganti dengan ip node dan port rpcnya sesuaikan yg di node**
##### 5. configurasi expolrer**

```
nano /etc/nginx/sites-enabled/<DOMAIN_EXPLORER>.conf
```

**ubah `<DOMAIN_EXPLORER>.conf` jadi domain explorer anda contoh `explorer.jembutmerah.dev`**

```
cp ~/explorer/ping.conf /etc/nginx/sites-enabled/<DOMAIN_EXPLORER>.conf
```

**Ubah nilai servername dari `_` domain explorer anda contoh `explorer.jembutmerah.dev`**
##### Test configurasi
```
nginx -t 
```
- ** Jika konfigurasinya benar seperti ini**
`nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`
`nginx: configuration file /etc/nginx/nginx.conf test is successful`
##### Install ssl sertifikat
```
sudo certbot --nginx --register-unsafely-without-email
sudo certbot --nginx --redirect

```

## Deploy explorer 
##### 1. Clone repo
```
cd ~
git clone https://github.com/AoiNode/explorer

```
##### 2. (Optional) Hapus semua konfigurasi yang ada
```
rm -rf ~/explorer/src/chains/mainnet/*

```
##### 3. Tambahkan konfig sendiri
```7
nano ~/explorer/src/chains/mainnet/<CHAIN_NAME>.json
```
- **Contoh konfig punya saya **
```
{
    "chain_name": "planq",
    "api": ["https://api-planq.aoinode.my.id", "https://rest.planq.network"],
    "rpc": ["https://rpc-planq.aoinode.my.id", "https://rpc.planq.network"],
    "snapshot_provider": "",
    "sdk_version": "0.46.3",
    "coin_type": "60",
    "min_tx_fee": "5000000000000000",
    "addr_prefix": "plq",
    "logo": "/logos/planq.png",
    "keplr_features": ["ibc-transfer", "ibc-go", "eth-address-gen", "eth-key-sign"],
    "assets": [{
        "base": "aplanq",
        "symbol": "planq",
        "exponent": "18",
        "coingecko_id": "",
        "logo": "/logos/planq.png"
    }]
}
```
- **>NOTE: Yang paling berpengaruh di sini api jika configurasi api sisanya optional ngasal juga gapapa wkwk, cuman untuk nilai `exponent` itu di ambil jumlah 0 dari base denomnya contoh `1 NTRN = untrn1000000` 0nya ada 6** 
##### 4. Build
```
cd ~/explorer
yarn && yarn build

```
Note* Jika error
```
yarn install --ignore-engines
```
Lalu
``` yarn && yarn build```
##### 5. Hosting
```
cp -r $HOME/explorer/dist/* /usr/share/nginx/html
sudo systemctl restart nginx

```
###### Special thanks to
**ping.pub** https://github.com/ping-pub/explorer# rpcapiexplorer
