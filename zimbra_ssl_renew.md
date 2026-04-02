# Zimbra SSL Sertifika Yenileme (Let's Encrypt)

## 📌 Senaryo

- Zimbra Mail Server
- Let's Encrypt SSL
- Certbot kullanılıyor
- Sertifika süresi dolmuş (expired)

---

## 🔍 1. Mevcut Sertifika Kontrolü

```bash
su - zimbra
zmcertmgr viewdeployedcrt
```

---

## 🔍 2. Certbot Sertifikalarını Kontrol Et

```bash
certbot certificates
```

---

## ⚠️ 3. Sorun: Port 80 Kullanımda

Problem:

```
Problem binding to port 80
```

Sebep:

- Zimbra servisleri port 80’i kullanıyor

---

## 🛠️ 4. Çözüm: Zimbra Servislerini Durdur

```bash
su - zimbra -c "zmcontrol stop"
```

Port kontrolü:

```bash
ss -ltnp | grep ':80 '
```

---

## 🔄 5. Sertifika Yenileme

```bash
certbot renew
```

---

## 📂 6. Sertifikaları Zimbra Dizine Kopyala

```bash
cp /etc/letsencrypt/live/mail.bilginay.com.tr/cert.pem /opt/zimbra/ssl/zimbra/commercial/
cp /etc/letsencrypt/live/mail.bilginay.com.tr/chain.pem /opt/zimbra/ssl/zimbra/commercial/
cp /etc/letsencrypt/live/mail.bilginay.com.tr/privkey.pem /opt/zimbra/ssl/zimbra/commercial/commercial.key
```

---

## 🔐 7. Yetkileri Düzenle

```bash
chown zimbra:zimbra /opt/zimbra/ssl/zimbra/commercial/*
chmod 640 /opt/zimbra/ssl/zimbra/commercial/cert.pem
chmod 640 /opt/zimbra/ssl/zimbra/commercial/chain.pem
chmod 600 /opt/zimbra/ssl/zimbra/commercial/commercial.key
```

---

## 📜 8. Root CA Sertifikasını İndir

```bash
curl -L https://letsencrypt.org/certs/isrgrootx1.pem -o /opt/zimbra/ssl/zimbra/commercial/isrgrootx1.pem
chown zimbra:zimbra /opt/zimbra/ssl/zimbra/commercial/isrgrootx1.pem
chmod 644 /opt/zimbra/ssl/zimbra/commercial/isrgrootx1.pem
```

---

## 🔗 9. CA Chain Dosyasını Oluştur

```bash
cat /opt/zimbra/ssl/zimbra/commercial/chain.pem \
    /opt/zimbra/ssl/zimbra/commercial/isrgrootx1.pem \
    > /opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt

chown zimbra:zimbra /opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt
chmod 644 /opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt
```

---

## ✅ 10. Sertifika Doğrulama

```bash
su - zimbra -c "/opt/zimbra/bin/zmcertmgr verifycrt comm \
/opt/zimbra/ssl/zimbra/commercial/commercial.key \
/opt/zimbra/ssl/zimbra/commercial/cert.pem \
/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt"
```

---

## 🚀 11. Sertifikayı Deploy Et

```bash
su - zimbra -c "/opt/zimbra/bin/zmcertmgr deploycrt comm \
/opt/zimbra/ssl/zimbra/commercial/cert.pem \
/opt/zimbra/ssl/zimbra/commercial/commercial_ca.crt"
```

---

## ▶️ 12. Zimbra Servislerini Başlat

```bash
su - zimbra -c "zmcontrol start"
```

---

## 🔎 13. Son Kontrol

```bash
su - zimbra -c "zmcertmgr viewdeployedcrt"
```

👉 Yeni `notAfter` tarihi görünmeli.

---

## 🎯 Sonuç

- Sertifika yenilendi
- Zimbra’ya deploy edildi
- SSL hataları giderildi
