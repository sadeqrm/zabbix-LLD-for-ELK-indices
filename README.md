<div dir="rtl" style="text-align:right">

<h1 align="center">📊 Zabbix LLD for ELK Indices</h1>
<p align="center">Low-level discovery (LLD) برای ایندکس‌های Elasticsearch و مانیتور کردن سایز/تعداد داکیومنت در Zabbix</p>

---

## 📑 فهرست مطالب
- 🔹 مرحله ۱ – اسکریپت کشف ایندکس‌ها
- 🔹 مرحله ۲ – معرفی به Zabbix Agent
- 🔹 مرحله ۳ – تنظیمات داخل Zabbix UI
- 📌 نکات مهم
- 📝 خلاصه‌نویسی
- ✅ چک لیست سریع

---

## 🔹 مرحله ۱ – اسکریپت کشف ایندکس‌ها
ابتدا یک اسکریپت به نام /usr/local/bin/es_index_discovery.sh ایجاد کنید:

> ⚠️ **هشدار مهم:**  
> قبل از اجرای هر دستور `curl`، مطمئن شوید که مقادیر زیر را با اطلاعات واقعی خود جایگزین کرده‌اید:  
> - `ES_HOST` : آدرس سرور Elasticsearch شما  
> - `ES_USER` : نام کاربری  
> - `ES_PASS` : رمز عبور


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
  echo -n "  { "{#ESINDEX}":"$idx" }"
  first=0
done
echo '] }'
```

---

## 🔹 مرحله ۲ – معرفی به Zabbix Agent
فایل تنظیمات زیر را در مسیر /etc/zabbix/zabbix_agentd.d/elasticsearch.conf اضافه کنید:

<div style="background-color:#fff3cd; color:#856404; border:1px solid #ffeeba; padding:10px; border-radius:5px;">
> ⚠️ **هشدار مهم:**  
> تنظیمات curl خودتان رو مطابق نام کاربری و رمز عبور ElasticSearch قرار دهید و از آدرس مربوط به سرورتان استفاده کنید.  
```ini
UserParameter=es.indices.discovery,/usr/local/bin/es_index_discovery.sh
UserParameter=es.index.size[*],curl -s -u rakhshani:123456 "http://192.168.44.121:9200/_cat/indices/$1?h=store.size" | awk '{print $1}'
```
سپس ریستارت سرویس Zabbix Agent:
```bash
systemctl restart zabbix-agent
```

---

## 🔹 مرحله ۳ – تنظیمات داخل Zabbix UI
1. وارد بخش Configuration → Templates شوید.
2. یک Template جدید با نام Template Elasticsearch Indices بسازید.
3. در بخش Discovery rules:
- Name: Elasticsearch Index Discovery
- Key: es.indices.discovery
- Type: Zabbix agent
- Update interval: 1h
4. سپس در بخش Item prototypes:
- Name: Index {#ESINDEX} size
- Key: es.index.size[{#ESINDEX}]
- Type: Zabbix agent
- Units: B یا MB

---

## 📌 نکات مهم
- اطمینان حاصل کنید که کاربر ELK دسترسی خواندن روی _cat/indices داشته باشد.
- مقادیر خروجی store.size ممکن است بر اساس نسخه Elasticsearch متفاوت باشد.
- برای بهینه‌سازی می‌توانید از HTTPs و کاربر اختصاصی با حداقل دسترسی استفاده کنید.

---

## 📝 خلاصه‌نویسی
- با یک اسکریپت Bash ایندکس‌ها را کشف کردیم.
- آن را به عنوان UserParameter به Zabbix Agent معرفی کردیم.
- در Zabbix UI یک Template ساختیم و Discovery Rule + Item Prototype تنظیم کردیم.
- از این پس Zabbix به صورت خودکار ایندکس‌های جدید را کشف کرده و سایز آنها را مانیتور می‌کند.

---

## ✅ چک لیست سریع
- [x] اسکریپت es_index_discovery.sh ساخته شد
- [x] سطح دسترسی اجرا داده شد (chmod +x)
- [x] UserParameterها در Agent تعریف شد
- [x] Zabbix Agent ریستارت شد
- [x] Template در Zabbix ساخته شد
- [x] Discovery Rule و Item Prototype اضافه شد

</div>
