# SIEM Home Lab: Log Toplama ve Alarm Kuralları

Docker ile kurulan bir ELK Stack (Elasticsearch, Logstash, Kibana) laboratuvarı.
Linux `auth.log` kayıtlarını Logstash ile ayrıştırıp Elasticsearch'e alır; Windows VM'den
Sysmon ve Security loglarını Winlogbeat ile toplar; Kibana'da dashboard ve tespit kuralları kurar.

**Teknolojiler:** Elasticsearch / Logstash / Kibana 8.13, Docker Compose, Sysmon, Winlogbeat, KQL, Grok

## Mimari

```
auth.log ───────────────► Logstash (grok + date) ──┐
Windows VM (Sysmon) ──► Winlogbeat ──► Logstash ───┼──► Elasticsearch ──► Kibana
                                                   │       (lab-auth-*, lab-winlogbeat-*)
```

## Gereksinimler

- Docker ve Docker Compose, en az 4 GB boş RAM
- (İsteğe bağlı) Windows 10/11 sanal makinesi: Sysmon + Winlogbeat için

## Kurulum

```bash
# 1) Stack'i başlat
docker compose up -d

# 2) Elasticsearch hazır olana kadar bekle (1-2 dk), sonra:
curl http://localhost:9200

# 3) Veri görünümlerini oluştur
./scripts/setup_kibana.sh

# 4) Verinin geldiğini doğrula
./scripts/verify.sh

# 5) Kibana: http://localhost:5601  -> Discover -> lab-auth-*
```

> Örnek `auth.log`'un tarihleri **4 Ekim 2026**'dır. Kibana'da zaman aralığını buna göre
> (ör. "Last 1 year" ya da mutlak tarih) ayarlayın.

## Windows / Sysmon Kısmı (isteğe bağlı)

1. Windows VM'e [Sysmon](https://learn.microsoft.com/sysinternals/downloads/sysmon) kurun
   (örnek yapılandırma: SwiftOnSecurity `sysmon-config`).
2. `winlogbeat/winlogbeat.yml` içindeki `<HOST_IP>` değerini Docker host IP'si ile değiştirip Winlogbeat'i servis olarak çalıştırın.
3. Kibana'da `lab-winlogbeat-*` veri görünümünde olayları görün.

## Tespit Kuralları

Beş kural `detection-rules/README.md` içinde KQL sorgusu, MITRE ATT&CK eşleştirmesi ve
doğrulama adımlarıyla belgelenmiştir: SSH brute-force, başarısız girişlerden sonra başarılı giriş,
encoded PowerShell, certutil ile dosya indirme, yeni yerel kullanıcı oluşturma.

## Önerilen Dashboard

Kibana > Dashboards > Create ile şu panelleri ekleyin ve ekran görüntüsünü `docs/` altına koyun:

- Zaman içinde başarılı / başarısız giriş (çizgi grafik)
- En çok başarısız deneme yapan 10 kaynak IP (tablo veya çubuk)
- En çok denenen kullanıcı adları
- Sysmon: en sık çalışan süreçler, şüpheli komut satırları

## Güvenlik Notları

- `xpack.security` laboratuvar kolaylığı için **kapalıdır**; servisler yalnızca `127.0.0.1`'e açılır.
  Bu yapıyı internete açmayın.
- Örnek log sentetiktir (RFC 5737 IP aralığı).
- Sysmon tetikleme komutları yalnızca kendi sanal makinenizde çalıştırılmalıdır.

## Sınırlar ve Sonraki Adımlar

- Tek düğümlü, güvenliği kapalı bir lab; üretim mimarisi değildir.
- Sonraki adımlar: Elastic Security detection engine kuralları, Sigma kurallarının dönüştürülmesi, alarm bildirimi (e-posta/Slack).
