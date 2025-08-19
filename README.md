<h1 align="center">📊 Zabbix LLD for ELK Indices</h1>
<p align="center">Low-level discovery (LLD) برای مانیتورینگ ایندکس‌های Elasticsearch با Zabbix</p>

---

## 📑 فهرست مطالب
1. [🔎 معرفی](#-معرفی)  
2. [🔹 مرحله ۱ – اسکریپت کشف ایندکس‌ها](#-مرحله-۱--اسکریپت-کشف-ایندکسها)  
3. [🔹 مرحله ۲ – معرفی به Zabbix Agent](#-مرحله-۲--معرفی-به-zabbix-agent)  
4. [🔹 مرحله ۳ – تنظیمات در Zabbix UI](#-مرحله-۳--تنظیمات-در-zabbix-ui)  
5. [🗺️ دیاگرام معماری](#️-دیاگرام-معماری)  
6. [📌 جمع‌بندی](#-جمع‌بندی)  
7. [✅ چک‌لیست سریع](#-چکلیست-سریع)  

---

## 🔎 معرفی
در سیستم‌های **ELK (Elasticsearch, Logstash, Kibana)**، ایندکس‌ها به‌طور پیوسته ساخته می‌شوند و مدیریت سایز آن‌ها بسیار مهم است.  
با کمک **Zabbix LLD (Low Level Discovery)** می‌توان ایندکس‌ها را به صورت خودکار کشف کرد و اندازه آن‌ها را مانیتور نمود.  

---

## 🔹 مرحله ۱ – اسکریپت کشف ایندکس‌ها
📌 مسیر پیشنهادی:  
`/usr/local/bin/es_index_discovery.sh`

```bash
#!/bin/bash

ES_HOST="192.168.44.121:9200"
ES_USER="rakhshani"
ES_PASS="123456"
TODAY=$(date +%Y.%m.%d)

indices=$(curl -s -u $ES_USER:$ES_PASS "http://$ES_HOST/_cat/indices?h=index" | grep "$TODAY")

echo '{ "data":['
first=1
for idx in $indices; do
  if [ $first -eq 0 ]; then
    echo ','
  fi
  echo -n "  { \"{#ESINDEX}\":\"$idx\" }"
  first=0
done
echo '] }'
