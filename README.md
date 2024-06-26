# Docker
```bash
sudo apt update && \
sudo apt install curl git docker.io -y
```

## Repoyu Klonlama
```
git clone https://github.com/koltigin/Cosmos-Snapshots.git && cd Cosmos-Snapshots
```

## snapshots Klasörü ve Snapshot Alınacak Node'un Klasörünü Oluşturma 
Burada OKP4 dosyası oluşturulmuştur.
```
mkdir -p $HOME/snapshots/testnet/okp4
```

## Snapshot Alınacak Node'un Log Dosyasını Oluşturma 
Burada OKP4 dosyası oluşturulmuştur.
```
touch $HOME/snapshots/testnet/okp4/okp4_log.txt
```

## Nginx Kurulumu
```bash
sudo apt-get install nginx -y
sudo unlink /etc/nginx/sites-enabled/default
```

## Nginx'i Ayarlama ve Başlatma
Aşağıdaki kodla snapshot yayınlayacağımız sitenin dizinini belirliyoruz. Burada dizin root içerisinde bulunan snapshots klasörü dizin olacak.
```bash
sudo tee <<EOF >/dev/null /etc/nginx/sites-available/file-server.conf
server {
    listen       80;
    client_max_body_size 5G;
    proxy_max_temp_file_size 0;
    root /root/snapshots/;
    autoindex on;
}
EOF
sudo ln -s /etc/nginx/sites-available/file-server.conf /etc/nginx/sites-enabled/file-server.conf
chmod o+x /root/snapshot
chmod o+x /root/
service nginx restart
```

### Alternatif: Docker ile Nginx'i Başlatma
Yukarıdaki yaılandırma yerine docker ile yapılandırma yapmak isterseniz.
```bash
cd $HOME; \
docker run --name snapshots \
--restart always \
-v $(pwd)/default.conf:/etc/nginx/conf.d/default.conf \
-v $(pwd)/snapshots/:/root/ \
-p 80:80 \
-d nginx
```

## `example_snapshot.sh` dosyasını `okp4_snapshot.sh` olarak kopyalıyoruz
```bash
cp $HOME/Cosmos-Snapshots/scripts/example_snapshot.sh $HOME/Cosmos-Snapshots/scripts/okp4_snapshot.sh
```

## `okp4_snapshot.sh` Dosyasında Değişkenleri Ayarlama
```
CHAIN_ID="okp4-nemeton-1"
SNAP_PATH="$HOME/snapshots/testnet/okp4"
LOG_PATH="$HOME/snapshots/testnet/okp4/okp4_log.txt"
DATA_PATH="$HOME/.okp4d/data/"
SERVICE_NAME="okp4d.service"
```

## Yeni Snapshot Almak için Scripti Çalıştırma
Öncelikle dosyaya çalıştırma izinini veriyoruz.
```
chmod +x ./Cosmos-Snapshots/scripts/okp4_snapshot.sh
```

Son olarak scripti çalıştırıyoruz.
```
./Cosmos-Snapshots/scripts/okp4_snapshot.sh
```

## Snapshotu Kontrol Etme  
Snapshot alma işlemi tamamlandığında ilgili dosyayı
`/snapshots/okp4/` dosyası içerisinde görebilirsiniz.

## Snapshot Alınmasını Otomatikleştirme
Öncelikle cron yükleyip aktif hale getiriyoruz.
```
sudo apt install cron
sudo systemctl enable cron
```
Aşağıdaki kod ile crontab doyasını düzenlemek için açıyoruz.
```
crontab -e
```

Aşağıdaki kodu dosyaya ekleyip ctrl x, y ve enter diyerek kaydediyoruz.
```cron
# her gun saat 00:00'da baslat
0 0 * * * /bin/bash -c /root/Cosmos-Snapshots/scripts/okp4_snapshot.sh
```

Zaman ayarlamasının nasıl olacağını aşağıdan görebilirsiniz.
```cron
# İş tanımı örneği:
# .---------------- dakika (0 - 59)
# |  .------------- saat (0 - 23)
# |  |  .---------- gün (1 - 31)
# |  |  |  .------- ay (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- gün (0 - 6) (Pazar=0 ya da 7 olacak) YA DA yazıyla şu şekilde sun,mon,tue,wed,thu,fri,sat olarak belirtilecek.
# |  |  |  |  |
# 0  0  *  *  * /bin/bash -c /root/Cosmos-Snapshots/scripts/okp4_snapshot.sh
```
