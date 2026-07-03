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

## 🚀 Aşama 2 -> 3: Laboratuvar Aşaması - Keşifçi Veri Analizi (EDA) ve Temel Model Denemeleri

**Tarih:** 01 Temmuz 2026  
**Odak Noktası:** Jupyter Notebook üzerinde veri setini tanıma, görselleştirme (EDA) ve algoritmaları test etme (Prototipleme / Mutfak Süreci).

### 1. Yapılan İşlemler ve Mühendislik Mantığı

#### 📓 Neden Python Dosyalarından (`.py`) Tekrar Jupyter Notebook'a (`.ipynb`) Geçildi?
- **İşlem:** Projeye `notebook/` klasörü eklendi; veri seti (`data/stud.csv`) konuldu ve `1 . EDA STUDENT PERFORMANCE .ipynb` ile `2. MODEL TRAINING.ipynb` dosyaları oluşturuldu.
- **Neden Yapıldı? (Laboratuvar vs. Üretim Ayrımı):**
  Kurumsal makine öğrenmesi süreçleri iki net evreden oluşur:
  1. **Laboratuvar / Mutfak Evresi (`.ipynb`):** Veri seti ilk kez elimize ulaştığında doğrudan `src/components/` altındaki üretim kodlarını yazamayız. Önce veride kayıp değer (missing value) var mı, aykırı değerler neler, hedef değişken (Target Variable) nasıl dağılmış ve hangi özellikler (features) birbiriyle korele görmek gerekir. Grafikler çizmek, anlık hipotezler denemek ve ilk model kıyaslamalarını (Baseline Model Training) yapmak için Jupyter Notebook en hızlı geri bildirimi veren ortamdır.
  2. **Üretim / Production Evresi (`.py`):** Notebook üzerinde tüm formüller ve modeller test edilip başarıya ulaştıktan sonra bu kodlar notebook'ta **bırakılmaz**. Modüler hale getirilip `src/components/` altındaki `.py` dosyalarına taşınır.

#### 📦 Bağımlılıkların Güncellenmesi (`requirements.txt`)
- **İşlem:** Görselleştirme ve makine öğrenmesi algoritmaları için `matplotlib`, `scikit-learn`, `catboost`, `xgboost`, `Flask` paketleri eklendi.
- **Neden Yapıldı?** EDA aşamasında veri dağılımlarını görmek için `matplotlib/seaborn`, temel modelleri kurup denemek için ise `scikit-learn` ve ağaç tabanlı (boosting) algoritmalar (`catboost`, `xgboost`) gerekir.

---

### 💡 Kıdemli Mühendis Tavsiyesi / Kas Hafızası Kuralı
- Eğitmenden kodu ve veri setini GitHub'dan çekmek projeye dosyaları hızlıca getirmek için harika bir adımdır. Ancak gerçek öğrenme ve **klavye/kas hafızası** bu hücreleri sadece "Shift+Enter" ile çalıştırmakla gelişmez.
- **Yarınki Tekrar Stratejisi:** Kendi boş bir notebook dosyanı (`benim_eda_denemem.ipynb` gibi) açıp Pandas fonksiyonlarını (`pd.read_csv()`, `.isnull().sum()`, `.describe()`, `sns.histplot()`, `train_test_split()`) bizzat bakarak kendi parmaklarınla yazmalısın. Bir veriyi ön işlerken neden o adımı attığını hücrelerin tepesine Markdown notları düşerek pekiştirmek seni gerçek bir kıdemli mühendis yapar.

---

## 🚀 Aşama 3 -> 4: Veri Yutma Bileşeni (`DataIngestion` Component & Artifact Mimarisi)

**Tarih:** 02 Temmuz 2026  
**Odak Noktası:** Notebook (`.ipynb`) ortamında test edilen veri okuma ve ayırma işlemlerinin üretim standartlarında çalışan modüler bir Python bileşenine (`data_ingestion.py`) taşınması.

### 1. Yapılan İşlemler ve Mühendislik Mantığı

#### ⚙️ Yapılandırma Yönetimi İçin `@dataclass` Kullanımı (`DataIngestionConfig`)
- **İşlem:** Verilerin nereye kaydedileceğini belirten `DataIngestionConfig` sınıfı `@dataclass` dekoratörü ile tanımlandı.
- **Neden Yapıldı?** Veri yutma modülü çalışırken ham verinin (`data.csv`), eğitim setinin (`train.csv`) ve test setinin (`test.csv`) hangi klasöre çıkacağını bilmelidir. Python'da sabit ayarları ve yolları tutmak için standart `__init__` yazmak yerine `@dataclass` kullanmak endüstri standardı ve en temiz (Clean Code) yaklaşımdır.

#### 🏗️ `artifacts/` Klasörü ve Veri Akışı
- **İşlem:** Ham veri (`stud.csv`) okundu, `os.makedirs` ile `artifacts/` klasörü açıldı ve veriler %80 Eğitim (`train.csv`) - %20 Test (`test.csv`) olarak ayrılarak bu klasöre kaydedildi.
- **Neden Yapıldı? (Modüler Pipeline Temeli):** Üretim ortamında bir sonraki bileşen olan `DataTransformation` (Veri Dönüştürme), ham veriye sıfırdan gitmek yerine doğrudan `DataIngestion` modülünün ürettiği ve `artifacts/` içine bıraktığı `train.csv` ve `test.csv` dosyalarını okuyarak işe başlayacaktır. Bileşenlerin bu şekilde **girdi-çıktı dosyaları (`artifacts`) üzerinden haberleşmesi**, boru hattının (pipeline) çökmeden parça parça test edilebilmesini sağlar.

#### 🛡️ Hata Yakalama ve İzlenebilirlik
- **İşlem:** Veri yutma sürecinin her kritik adımı (`logging.info`) ile kayıt altına alındı ve `try-except` bloğu ile sarmalanarak olası çökmeler `CustomException(e, sys)` üzerinden fırlatıldı.

---

### 💡 Kıdemli Mühendis Gözüyle Kod İncelemesi (Code Review & Kritik Bulgular)

Bu aşamada yazılan `src/components/data_ingestion.py` dosyasında bir mühendisin mutlaka fark edip düzeltmesi gereken 3 önemli nokta bulunuyor:

1. **Kritik Yazım Hatası (Typo & AttributeError Riski):**  
   [data_ingestion.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/data_ingestion.py#L21) dosyasının 21. ve 47. satırlarında fonksiyon ismi `initiate_data_ingesiton` olarak yazılmış (ingestion yerine **ingesiton**).  
   *Neden tehlikeli?* Bir sonraki aşamada `train_pipeline.py` bu sınıfı çağırıp doğru kelimelerle `obj.initiate_data_ingestion()` dediği an sistem çökecek ve `AttributeError` fırlatacaktır. Kodunu hemen düzeltmelisin.

2. **PEP 8 Return Formatı:**  
   [data_ingestion.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/data_ingestion.py#L37) dosyasında `return(...)` bitişik yazıldığı için fonksiyon çağrısı gibi görünmektedir. Python'da `return` anahtar kelimesinden sonra bir boşluk bırakılmalıdır:
   ```python
   return (
       self.ingestion_config.train_data_path,
       self.ingestion_config.test_data_path,
       self.ingestion_config.raw_data_path
   )
   ```

3. **Gereksiz Importlar:**  
   [data_ingestion.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/data_ingestion.py#L2) dosyasındaki `import random` ve `import numpy as np` kütüphaneleri bu modül içinde hiç kullanılmamaktadır. Temiz kod prensibi gereği silinmelidir.

---

## 🚀 Aşama 4 -> 5: Veri Dönüştürme (`DataTransformation` Component & Scikit-Learn Pipelines)

**Tarih:** 02 Temmuz 2026  
**Odak Noktası:** Sayısal ve kategorik sütunların üretim standartlarında ön işlenmesi (Preprocessing), Scikit-Learn `Pipeline` / `ColumnTransformer` mimarisi ve ön işleme nesnesinin (`preprocessor.pkl`) serileştirilerek disk üzerine kaydedilmesi.

### 1. Yapılan İşlemler ve Mühendislik Mantığı

#### 🔀 Sayısal ve Kategorik Boru Hatları (`num_pipeline` & `cat_pipeline`)
- **İşlem:** Sayısal değişkenler için `SimpleImputer(strategy="median")` + `StandardScaler()`; kategorik değişkenler için `SimpleImputer(strategy="most_frequent")` + `OneHotEncoder()` + `StandardScaler(with_mean=False)` boru hatları yazıldı ve `ColumnTransformer` ile birleştirildi.
- **Neden Yapıldı?**
  1. **Median İle Doldurma:** Sayısal verilerde aykırı değerler (outliers) ortalamayı (`mean`) saptırır. Medyan ise aykırı değerlere karşı dayanıklıdır (robust).
  2. **`with_mean=False` Gerekçesi:** `OneHotEncoder` seyrek matrisler (sparse matrix, bol sıfırlı yapılar) üretir. Eğer `StandardScaler` içinde `with_mean=True` bırakılırsa seyrek matrisin sıfırları ortalama çıkarılarak dolgun matrise dönüşür, RAM'i kilitler ve sistemi çökertebilir.

#### 🛡️ Veri Sızıntısını (Data Leakage) Önleme: `fit_transform` vs `transform`
- **İşlem:** Eğitim verisine (`train_df`) `preprocessing_obj.fit_transform()` uygulanırken, test verisine (`test_df`) **sadece** `.transform()` uygulandı.
- **Neden Yapıldı? (En Kritik Mülakat Kuralı):** Eğer test verisine de `fit` edilseydi, medyan veya standart sapma hesaplanırken test verisinin bilgisi de kullanılmış olurdu. Bu duruma **Veri Sızıntısı (Data Leakage)** denir. Model yapay olarak yüksek başarı gösterir ama canlıya (production) çıkıldığında çuvallar!

#### 💾 Ön İşleme Nesnesinin Kaydedilmesi (`utils.save_object` & `pickle`)
- **İşlem:** Eğitilen `preprocessing_obj` (preprocessor), `src/utils.py` içindeki `save_object` fonksiyonu kullanılarak `artifacts/` klasörüne disk dosyası olarak kaydedildi.
- **Neden Yapıldı?** Canlı ortamda web sitemize (`Flask` app / `predict_pipeline.py`) bir öğrenci gelip verilerini girdiğinde, o verinin de eğitim aşamasındaki **aynı medyan ve ağırlıklarla** standartlaştırılması şarttır. Bu nesneyi (`preprocessor`) kaydetmek, tahmin boru hattının bel kemiğidir.

---

### 💡 Kıdemli Mühendis Gözüyle Kod İncelemesi (Code Review & Kritik Uyarılar)

Yazdığın `src/components/data_transformation.py` ve `src/utils.py` dosyalarında dikkat etmen gereken 2 kritik mühendislik notu:

1. **Önemli Dosya Adı Yazım Hatası (`proprocessor.pkl` vs `preprocessor.pkl`):**  
   [data_transformation.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/data_transformation.py#L19) dosyasının 19. satırına dikkat et:
   ```python
   # Mevcut hal:
   preprocessor_obj_file_path = os.path.join('artifacts',"proprocessor.pkl")
   ```
   Dosya adını kelime başındaki 'e' yerine 'o' ile **`proprocessor.pkl`** olarak kaydetmişsin.  
   *Tehlikesi ne?* İleride `predict_pipeline.py` yazarken standart isme alışıp `preprocessor.pkl` okumaya çalıştığında `FileNotFoundError` alacaksın. Dosya adını `preprocessor.pkl` olarak düzeltmelisin.

2. **Kullanılmayan Kütüphane (`import dill`):**  
   [utils.py](file:///c:/Users/deniz/PYTHON/mlproject/src/utils.py#L6) dosyasının 6. satırında `import dill` eklenmiş fakat nesne kaydetme/yükleme işlemlerinde standart `pickle` kullanılmıştır. Kullanılmayan paketi kaldırmak kodunu sadeleştirir.

---
