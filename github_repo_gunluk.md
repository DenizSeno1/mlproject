# 🛠️ Uçtan Uca Makine Öğrenmesi Projesi - Geliştirme Günlüğü (ML Arşiv)

Bu dosya, Krish Naik ile uçtan uca makine öğrenmesi (End-to-End Machine Learning) projesi geliştirirken adım adım atılan mühendislik adımlarını, mimari kararları ve "neden" sorularının cevaplarını barındıran kişisel bir mühendislik alet çantasıdır (Toolbox & Logbook).

---

## 🚀 Aşama 0 -> 1: Proje Kurulumu ve Paketleme Mimarisi (Project Setup & Packaging)

**Tarih:** 01 Temmuz 2026  
**Odak Noktası:** Sanal ortam kurulumu, Git versiyon kontrolü, standart Python paketi (`package`) yapısı ve otomatik bağımlılık yönetimi.

### 1. Yapılan İşlemler ve Mühendislik Mantığı

#### 📦 Sanal Ortam (`venv`) ve `.gitignore`
- **İşlem:** Projeye izole bir sanal ortam (`venv`) kuruldu ve bir `.gitignore` dosyası eklendi.
- **Neden Yapıldı?** Her projenin kütüphane versiyonları (Örn: Pandas 1.x vs 2.x) çakışabilir. Projeleri sistem Python'undan izole etmek zorunludur. Ayrıca `venv/`, `__pycache__/` ve `.egg-info/` gibi platforma özel/geçici dosyaların Git repozitorisini şişirmesi `.gitignore` ile engellendi.

#### 🏗️ `src/` Klasör Mimarisi ve `__init__.py`
- **İşlem:** Kodların ana dizine dağılmasını önlemek amacıyla `src/` klasörü açıldı ve içerisine boş bir `__init__.py` dosyası konuldu.
- **Neden Yapıldı?** Python bir klasör içerisinde `__init__.py` gördüğünde o dizini standart bir **modül/paket (package)** olarak algılar. Bu mimari sayesinde projenin herhangi bir noktasından `from src.components... import ...` şeklinde temiz import işlemleri yapılabilir.

#### ⚙️ `setup.py` ve `requirements.txt` (`-e .` Mantığı)
- **İşlem:** Projenin meta verilerini (yazar, versiyon vb.) ve bağımlılıklarını yöneten `setup.py` yazıldı. `requirements.txt` dosyasının en altına `-e .` eklendi.
- **Neden Yapıldı?**
  1. **`setup.py` ile Paketleme:** `find_packages()` metodu, `src/` altındaki tüm paketleri bulur ve projemizi sistemimizde kurulabilir bir kütüphaneye dönüştürür.
  2. **`-e .` (Editable Install):** Terminalde `pip install -r requirements.txt` çalıştırıldığında pip en alttaki `-e .` komutunu okur ve otomatik olarak ana dizindeki `setup.py` dosyasını tetikler. Proje geliştirme (`editable`) modunda kurulur (`mlproject.egg-info` klasörü oluşur). Böylece kod üzerinde yaptığımız her değişiklik anında sisteme yansır.
  3. **`get_requirements()` Fonksiyonu:** `requirements.txt` okunurken `-e .` satırı listeden silinir (`requirements.remove('-e .')`). Çünkü PyPI sunucularında `-e .` adında bir kütüphane yoktur; silinmezse pip kurulumda hata verir.

---

### 💡 Kıdemli Mühendis Notu / Clean Code Hatırlatması
- List comprehension veya genel Python kodu yazılırken **PEP 8** standartlarına uyulmalı, operatör ve anahtar kelimelerin etrafında birer boşluk bırakılmalıdır:
  ```python
  # Doğru PEP 8 Kullanımı:
  requirements = [req.replace("\n", "") for req in requirements]
  ```

---
