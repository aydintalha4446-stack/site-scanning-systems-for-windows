import os
import sys
import time
import random
import threading
import requests

# Windows terminalinde Türkçe karakter ve konsol uyumluluğu için UTF-8 ayarı
if sys.platform.startswith("win"):
    try:
        sys.stdout.reconfigure(encoding='utf-8')
    except AttributeError:
        pass

# --- AYARLAR ---
# Buraya Bugcrowd veya platform kullanıcı kimliğini yazabilirsin
USER_ID = "BURAYA_KULLANICI_ID_YAZ" 
REPORT_FILE = "site_tarar_rapor.txt"
MAX_THREADS = 100  # Aynı anda taranacak site sayısı
            
headers = {
    "User-Agent": f"Bugcrowd-Researcher-{USER_ID}",
    "X-Purpose": "Security Research - White Hat",
    "Accept": "text/html"   
}
                
# Program başlarken rapor dosyasını hazırla
with open(REPORT_FILE, "a", encoding="utf-8") as f:
    f.write(f"\n--- TURBO TARAMA BASLATILDI (WINDOWS): {time.ctime()} ---\n")
             
def generate_random_ip():
    return f"{random.randint(1, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}.{random.randint(0, 255)}"
                
def log_vulnerability(url, reason):
    with open(REPORT_FILE, "a", encoding="utf-8") as f:
        f.write(f"[!] ZAFİYET: {url} | Neden: {reason} | {time.ctime()}\n")
    
    # Windows konsolunda dikkat çekmesi için bip sesi ve ekrana yazdırma
    print(f"\a\n\n>>> [BULDUM!] {url} - {reason} <<<\n")
        
def scanner_worker():
    """Sürekli rastgele siteleri tarayan işçi fonksiyonu."""
    while True:
        target_ip = generate_random_ip()
        target_url = f"http://{target_ip}"
        try:
            # 2 saniye içinde cevap vermezse diğer siteye geç (Hız için kritik)
            response = requests.get(f"{target_url}/debug-test-123", headers=headers, timeout=2)

            # Django and Hassas Veri Göstergeleri
            indicators = [
                "DisallowedHost",
                "django.views.debug",
                "TemplateDoesNotExist",
                "LOCAL_ADDR",
                "ENVIRONMENT"
            ]
    
            if any(ind in response.text for ind in indicators):
                log_vulnerability(target_url, "Django Debug Mode / Hassas Veri İfşası")

            # Ekstra: Admin paneli bulma denemesi
            if response.status_code == 200 and ("login" in response.url.lower() or "admin" in response.url.lower()):
                 pass
                 
        except:
            pass
    
# --- ANA PROGRAM ---
if __name__ == "__main__":
    print("="*50)
    print("   SITE TARAR v2.0 [TURBO MOD - WINDOWS] ")
    print(f"   AKTİF KANAL SAYISI: {MAX_THREADS}")
    print("   'Sürekli deneyen kaybetmez.'")
    print("="*50)

    # Kanalları (Threads) oluştur ve başlat
    threads = []
    for i in range(MAX_THREADS):
        t = threading.Thread(target=scanner_worker)
        t.daemon = True
        t.start()
        threads.append(t)
        if i % 10 == 0:
            print(f"[*] {i} kanal aktif hale getirildi...")
                
    print("\n[+] Tarama tam gaz devam ediyor... Durdurmak için: Ctrl+C")
                    
    try:
        while True:
            time.sleep(10) # Ana programı uyut ama alt kanallar çalışsın 
    except KeyboardInterrupt:
        print("\n\n[!] Turbo mod kapatılıyor. Rapor dosyasını kontrol et!")
