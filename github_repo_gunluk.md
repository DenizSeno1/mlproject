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

## 🚀 Aşama 1 -> 2: Proje Mimarisi, Loglama (Logging) ve Özel Hata Yönetimi (Exception Handling)

**Tarih:** 01 Temmuz 2026  
**Odak Noktası:** Modüler klasör yapısı (Components & Pipelines), izlenebilirlik için merkezi loglama sistemi ve production seviyesinde hata yakalama mekanizması.

### 1. Yapılan İşlemler ve Mühendislik Mantığı

#### 🗂️ Modüler Mimarinin Kurulması (`components/`, `pipeline/`, `utils.py`)
- **İşlem:** `src` altında `components` ve `pipeline` alt paketleri (içlerinde `__init__.py` ile birlikte) ve `utils.py` dosyası oluşturuldu.
- **Neden Yapıldı?** Tek bir Jupyter Notebook veya devasa bir `.py` dosyası içinde tüm ML adımlarını yazmak ("spagetti kod") enterprise projelerde kabul edilemez. Sorumlulukların Ayrılığı (*Separation of Concerns*) prensibi gereği:
  - **`components/`**: Verinin sisteme alınması (`data_ingestion.py`), dönüştürülmesi (`data_transformation.py`) ve modelin eğitilmesi (`model_trainer.py`) gibi bağımsız iş birimlerini tutar.
  - **`pipeline/`**: Bu bileşenleri uçtan uca bir sırada çalıştıracak olan eğitim (`train_pipeline.py`) ve tahmin (`predict_pipeline.py`) hatlarını barındırır.
  - **`utils.py`**: Model kaydetme, dosya okuma/yazma gibi tüm bileşenlerin ihtiyaç duyacağı ortak araçları sağlar.

#### 🛡️ Özel Hata Yönetimi (`src/exception.py` & `CustomException`)
- **İşlem:** Python'ın yerleşik `sys` kütüphanesi kullanılarak hatanın gerçekleştiği dosya ve satır numarasını otomatik çeken `CustomException` sınıfı yazıldı.
- **Neden Yapıldı?** Production (canlı) ortamında bir hata çıktığında sadece "ValueError" veya genel bir mesaj görmek ayıklamayı (debugging) imkansızlaştırır. `error_detail.exc_info()` fonksiyonu üzerinden execution traceback (`exc_tb`) çekilerek hatanın **hangi dosyanın (`co_filename`), kaçıncı satırında (`tb_lineno`) ve ne sebeple** oluştuğu standart bir mesaja dönüştürüldü.

#### 📝 Merkezi Loglama Sistemi (`src/logger.py`)
- **İşlem:** `logging` kütüphanesi yapılandırılarak çalışan kodun loglarını tarih-saat damgalı dosyalara (`logs/`) kaydeden sistem kuruldu.
- **Neden Yapıldı?** Otomatize edilmiş veri hatlarında (pipelines) terminal ekranı olmaz. Sistem gece 03:00'te çalıştığında hangi adımın başarılı olduğunu veya nerede çöktüğünü geriye dönük inceleyebilmek (audit & observability) için her işlemin loglanması şarttır.

---

### 💡 Kıdemli Mühendis Gözüyle Kod İncelemesi (Code Review & Kritik Bulgular)

Bu aşamada yazılan kodlarda Krish Naik'in eğitim videosunda da yer alan iki kritik nokta var. Bir mühendis olarak bunlara dikkat etmelisin:

1. **Gereksiz Import (Unused Import Risk):**  
   [logger.py](file:///c:/Users/deniz/PYTHON/mlproject/src/logger.py#L5) dosyasının 5. satırında `from src.exception import CustomException` import edilmiş ama dosya içinde hiç kullanılmamış. Gereksiz importlar kodun temizliğini bozar ve ileride dairesel bağımlılıklara (*circular import*) yol açabilir.

2. **Gizli Klasör/Dosya İsimlendirme Hatası (Klasik Video Hatası):**  
   [logger.py](file:///c:/Users/deniz/PYTHON/mlproject/src/logger.py#L8-L11) dosyasındaki şu satırları inceleyelim:
   ```python
   LOG_FILE = f"{datetime.now().strftime('%m_%d_%Y_%H_%M_%S')}.log"
   logs_path = os.path.join(os.getcwd(), "logs", LOG_FILE)
   os.makedirs(logs_path, exist_ok=True)
   LOG_FILE_PATH = os.path.join(logs_path, LOG_FILE)
   ```
   *Burada ne oluyor?* `logs_path` değişkeninin içine `LOG_FILE` eklendiği için `os.makedirs` komutu `logs/` altında dosya adı gibi görünen (`07_01_2026_16_41_10.log`) bir **klasör** oluşturur. Ardından bu klasörün içine aynı isimle dosya kaydeder!  
   İdeal ve endüstri standardı olan yaklaşım, `logs_path` değişkeninin sadece klasör yolunu tutmasıdır:
   ```python
   LOG_FILE = f"{datetime.now().strftime('%m_%d_%Y_%H_%M_%S')}.log"
   logs_path = os.path.join(os.getcwd(), "logs")  # Sadece klasör yolu
   os.makedirs(logs_path, exist_ok=True)

   LOG_FILE_PATH = os.path.join(logs_path, LOG_FILE)
   ```

---
