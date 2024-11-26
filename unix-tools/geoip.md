App que nos ofrece la localización de una ip:

yum install -y GeoIP
geoipupdate

$ geoiplookup -f /usr/share/GeoIP/GeoLiteCity.dat "70.113.45.187"
GeoIP City Edition, Rev 1: US, TX, Texas, Austin, 78744, 30.176001, -97.737297, 635, 512


La base de datos de ASNs tenemos que bajarla a mano:
https://download.maxmind.com/download/geoip/database/asnum/GeoIPASNum.dat.gz


Actualizar las bbdd de pais y ciudad cada dia:
ln -s /usr/bin/geoipupdate /etc/cron.daily/
