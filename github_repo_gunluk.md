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

## 🚀 Aşama 5 -> 6: Model Eğitimi, Hiperparametre Optimizasyonu ve Seçimi (`ModelTrainer` Component)

**Tarih:** 03 Temmuz 2026  
**Odak Noktası:** Dönüştürülmüş matrislerin (`train_arr`, `test_arr`) ayrıştırılması, 7 farklı regresyon algoritmasının `GridSearchCV` ile çapraz doğrulamaya (Cross-Validation) tabi tutularak en iyi hiperparametrelerinin bulunması, eşik değeri (Threshold) kontrolü ve kazanan modelin (`model.pkl`) canlı ortam için mühürlenmesi.

### 1. Yapılan İşlemler ve Mühendislik Mantığı (Derinlemesine Analiz)

#### 🧮 Matris Dilimleme (Array Slicing) ve Veri Hazırlığı
- **İşlem:** `data_transformation.py` modülünden gelen `train_array` ve `test_array` girdileri NumPy dilimleme kuralı ile `X_train`, `y_train`, `X_test`, `y_test` olarak ayrıştırıldı:
  ```python
  X_train, y_train = train_array[:, :-1], train_array[:, -1]
  ```
- **Mühendislik Mantığı (Neden?):** Bir önceki adımda (`DataTransformation`) bağımsız değişkenler (`input_features`) ile hedef değişken (`math_score`) `np.c_` metodu kullanılarak yan yana tek bir matrise yapıştırılmıştı. En sağdaki sütun (`-1`) hedef değişkenimizdir. Sütun isimlerinden bağımsız olarak saf NumPy indekslemesiyle veriyi ayırmak, bellek transferini hızlandırır ve veri boru hattında sütun kayması riskini engeller.

#### ⚔️ Çoklu Algoritma Arena Savaşı (Model Dictionary)
- **İşlem:** Tek bir modele bağlı kalmamak adına `Random Forest`, `Decision Tree`, `Gradient Boosting`, `Linear Regression`, `XGBRegressor`, `CatBoostRegressor` ve `AdaBoostRegressor` olmak üzere 7 farklı algoritma bir sözlük (`models`) içine yerleştirildi.
- **Mühendislik Mantığı (No Free Lunch Teoremi):** Makine öğrenmesinde "Her veri setinde en iyi çalışan tek bir süper algoritma vardır" diye bir kural yoktur (Buna literatürde *No Free Lunch Theorem* - Bedava Öğle Yemeği Yok Teoremi denir). Tablo verilerinde bazen basit bir Doğrusal Regresyon, bazen de derin ağaç tabanlı CatBoost en yüksek performansı verebilir. Bu yüzden profesyonel ardışık düzenlerde tüm adaylar aynı ringe çıkarılır.

#### 🎯 Hiperparametre Optimizasyonu ve Çapraz Doğrulama (`GridSearchCV` & `evaluate_models`)
- **İşlem:** `src/utils.py` içindeki `evaluate_models` fonksiyonunda her model için tanımlanan aday parametre uzayı (`params`) `GridSearchCV(model, para, cv=3)` ile test edildi.
- **Mühendislik Mantığı (`GridSearchCV` Arka Planda Ne Yapar?):**
  1. **Arama Uzayı:** Örneğin `Random Forest` için `n_estimators: [8, 16, 32, 64, 128, 256]` tanımlandığında 6 farklı ağaç sayısı kombinasyonu dener.
  2. **`cv=3` (3-Fold Cross Validation):** Veri setini 3 parçaya böler. 2 parçayla modeli eğitip 1 parçayla test eder. Bunu 3 kez döndürerek tesadüfi başarıları eler.
  3. **Yeniden Eğitim (Refit):** En iyi hiperparametre kombinasyonu (`best_params_`) bulunduktan sonra model `model.set_params(**gs.best_params_)` komutuyla güncellenir ve eğitim verisi üzerinde son halini alır.
  4. **Performans Metriği ($R^2$ Score):** Regresyon problemlerinde modelin bağımlı değişkeni açıklama yüzdesini gösteren $R^2$ (R-squared) başarı metriği üzerinden modeller kıyaslanır.

#### 🛡️ Kalite Kapısı ve Eşik Değeri Kontrolü (Quality Gate & Thresholding)
- **İşlem:** Arena savaşı sonucunda en yüksek $R^2$ skorunu alan model (`best_model_score`) belirlendi. Eğer skor %60'ın (`0.6`) altındaysa sistem `CustomException("No best model found")` fırlatarak çalışmayı durduracak şekilde programlandı.
- **Mühendislik Mantığı (Model Governance):** Canlı sisteme kötü bir model koymak, hiç model koymamaktan daha tehlikelidir (Örn: Yanlış kredi skoru veya savunma sanayiinde hatalı menzil tahmini). Eğer hiçbir algoritma %60 başarıyı geçemiyorsa veri setinde yetersizlik veya özellik mühendisliğinde (Feature Engineering) noksanlık var demektir. Bu durumda hatalı modelin `artifacts/model.pkl` olarak üretilmesine izin verilmez, sistem alarm verir.

#### 💾 Kazanan Şampiyonun Mühürlenmesi (`model.pkl`)
- **İşlem:** Kalite kapısını geçen şampiyon model `save_object` ile `artifacts/model.pkl` olarak diske yazıldı.
- **Mühendislik Mantığı:** Canlı tahmin servisi (`predict_pipeline.py`) devreye girdiğinde sıfırdan model eğitmez; milisaniyeler içinde sonuç vermek için doğrudan bu mühürlü `model.pkl` dosyasını belleğe yükleyerek çıkarım (inference) yapar.

---

### 💡 Kıdemli Mühendis Gözüyle Kod İncelemesi (Code Review & Clean Code)

Yazdığın `src/components/model_trainer.py` dosyasında bir mühendisin kod kalitesini (Clean Code) yükseltmek için dikkat etmesi gereken 3 bulgu:

1. **Kullanılmayan Ölü Importlar (Unused Imports):**  
   [model_trainer.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/model_trainer.py#L6-L7) dosyasının 6. ve 7. satırlarında:
   ```python
   from numpy import save
   from sklearn import preprocessing
   ```
   kütüphaneleri import edilmiştir ancak kodun hiçbir yerinde `save()` veya `preprocessing` kullanılmamaktadır. Linter uyarılarını temizlemek ve kodun bellek ayak izini azaltmak için bu satırları silmelisin.

2. **Sabit Sayıların Karar Mekanizmasına Gömülmesi (Magic Number Code Smell):**  
   [model_trainer.py](file:///c:/Users/deniz/PYTHON/mlproject/src/components/model_trainer.py#L98) dosyasında:
   ```python
   if best_model_score < 0.6:
   ```
   ifadesi yer almaktadır. İş kurallarındaki eşik değerleri doğrudan kodun ortasına sabit sayı (magic number) olarak yazmak yerine yapılandırma sınıfında (`ModelTrainerConfig`) tanımlanmalıdır:
   ```python
   @dataclass
   class ModelTrainerConfig:
       trained_model_file_path = os.path.join("artifacts", "model.pkl")
       expected_accuracy: float = 0.6  # Standart mühendislik yaklaşımı
   ```

3. **Yorum Satırında Bırakılmış Ölü Kodlar:**  
   `params` sözlüğü içerisinde `# 'loss': ['squared_error', ...]` gibi çok sayıda yoruma alınmış deneme satırı kalmış. Laboratuvar (`.ipynb`) aşamasında bu denemeler normalken, üretim modülleri (`.py`) depoya pushlanırken ya temizlenmeli ya da neden devredışı bırakıldığına dair teknik açıklama eklenmelidir.

---

## 🚀 Aşama 6 -> 7: Tahmin Boru Hattı (`PredictPipeline`) ve Flask Web Servisi (`app.py`)

**Tarih:** 03 Temmuz 2026  
**Odak Noktası:** Kullanıcı arayüzünden (`HTML`) gelen ham girdilerin Pandas DataFrame modeline dönüştürülmesi (`CustomData`), ön işleme (`preprocessor.pkl`) ve model (`model.pkl`) nesnelerinin uçtan uca çıkarım (inference) için bağlanması ve Flask tabanlı web servisinin entegrasyonu.

### 1. Yapılan İşlemler ve Mühendislik Mantığı (Derinlemesine Analiz)

#### 🌐 Ham Web Verisinin Yapılandırılması (`CustomData` Sınıfı)
- **İşlem:** Kullanıcının web formundan (`home.html`) girdiği cinsiyet, etnik köken, öğle yemeği tipi, okuma/yazma notları gibi ham verileri kapsülleyen `CustomData` sınıfı ve bu verileri 1 satırlık bir tabloya çeviren `get_data_as_data_frame()` metodu yazıldı.
- **Mühendislik Mantığı (Neden DataFrame'e Çevriliyor?):** Web formundan gelen veriler kelime (string) veya sayı (float) halindeki gevşek değişkenlerdir. Ancak bizim 4. aşamada eğittiğimiz `ColumnTransformer` (`preprocessor.pkl`) ve 5. aşamada eğittiğimiz makine öğrenmesi modeli (`model.pkl`), kesin sütun isimlerine (`gender`, `race_ethnicity` vb.) ve veri tiplerine sahip 2 boyutlu tabular yapılar (Pandas DataFrame) bekler. `CustomData` sınıfı, dış dünyadan gelen düzensiz web verisini modelin şemasına uyduran bir **Adaptör (Adapter Pattern)** görevi görür.

#### 🔮 Uçtan Uca Çıkarım Motoru (`PredictPipeline`)
- **İşlem:** `PredictPipeline` sınıfı içindeki `predict(features)` metodu ile önce `preprocessor.pkl` ve `model.pkl` diskten yüklendi. Veri önce `preprocessor.transform(features)` ile ölçeklendirildi, ardından `model.predict(data_scaled)` ile son matematik notu tahmini yapıldı.
- **Mühendislik Mantığı (Sıralı İşlem Bağımlılığı):** Canlı sistemde kullanıcı "Kız, Grup B, Yüksek Lisans ebeveyni, 72 Okuma Notu" girdiğinde bu metinleri model doğrudan anlayamaz. Verinin **tam olarak eğitimdeki medyanlar ve One-Hot encoding kurallarıyla** dönüştürülmesi şarttır. Bu yüzden önce `transform()`, sonra `predict()` çağrılır. (Not: Burada `fit_transform()` değil sadece `transform()` kullanılmasının sebebi yine veri sızıntısını ve şema bozulmasını önlemektir).

#### 🌍 Sunucu ve Rota Yönetimi (`app.py` & Flask Routing)
- **İşlem:** Flask mikro web framework'ü kullanılarak iki ana rota (`/` ve `/predictdata`) tanımlandı. `GET` isteği form sayfasını (`home.html`) ekrana basarken, `POST` isteği formdaki verileri toplayıp boru hattını tetikledi ve sonucu ekrana (`results`) yansıttı.
- **Mühendislik Mantığı:** Makine öğrenmesi modelleri kapalı kutu (black box) scriptler olarak kalamaz. Kullanıcıların veya başka yazılımların modele erişebilmesi için HTTP protokolü üzerinden hizmet veren bir API veya web arayüzü ile sarmalanması (serving) gerekir.

---

### 💡 Kıdemli Mühendis Gözüyle Kod İncelemesi (Code Review & Kritik Bulgular)

Yazdığın `app.py`, `src/pipeline/predict_pipeline.py` ve `templates/home.html` dosyalarını incelediğimde, projenin kaderini etkileyecek **3 çok kritik bulgu** yakaladım:

1. **Kritik Mantık Hatası: Çapraz Çekilen Okuma ve Yazma Notları (Input Swapping Bug):**  
   [app.py](file:///c:/Users/deniz/PYTHON/mlproject/app.py#L29-L30) dosyasının 29. ve 30. satırlarına çok dikkatli bak:
   ```python
   reading_score=float(request.form.get('writing_score')),
   writing_score=float(request.form.get('reading_score'))
   ```
   *Ne oluyor?* `reading_score` değişkenine HTML formundaki `writing_score` değerini; `writing_score` değişkenine ise `reading_score` değerini atıyorsun! Ayrıca [home.html](file:///c:/Users/deniz/PYTHON/mlproject/templates/home.html#L94-L101) dosyasında da etiket (`label`) ile `name` nitelikleri ters yazılmış.  
   *Tehlikesi ne?* Kullanıcı ekrandan "Okuma: 90, Yazma: 40" girdiğinde, model bunu "Okuma: 40, Yazma: 90" olarak algılayıp tamamen yanlış bir Matematik notu tahmin edecektir. Veri akışındaki isimler tutarlı hale getirilmelidir.

2. **Ciddi Performans Darboğazı: Her İsteğe Özel Disk Okuma (Disk I/O Anti-Pattern):**  
   [predict_pipeline.py](file:///c:/Users/deniz/PYTHON/mlproject/src/pipeline/predict_pipeline.py#L13-L18) dosyasında model ve ön işleyici `predict()` fonksiyonunun **içinde** yükleniyor:
   ```python
   def predict(self, features):
       model = load_object(file_path=model_path)
       preprocessor = load_object(file_path=preprocessor_path)
   ```
   *Neden anti-pattern?* Diskten dosya okuma (`I/O operation`), RAM'den veri okumaya kıyasla binlerce kat daha yavaştır. Web sitene saniyede 1000 kullanıcı girse, sistem saniyede 1000 kez diskten devasa pickle dosyası okumaya çalışır ve sunucu çöker (High Latency / Bottleneck).  
   *Doğru Mimari:* Modeller `PredictPipeline.__init__()` içinde (yani nesne ilk oluşturulduğunda bir kez) belleğe (RAM) yüklenmeli, `predict()` metodu sadece RAM'deki hazır modeli kullanmalıdır.

3. **Üretim Ortamı İçin Hata ve Log Temizliği (Debug Print Statements):**  
   `app.py` ve `predict_pipeline.py` içinde `print("Before Prediction")`, `print("After Loading")` gibi test amaçlı `print` komutları bırakılmış. Production kodlarında terminale `print` basmak yerine kurduğumuz merkezi `logging.info(...)` altyapısı kullanılmalı veya bu satırlar silinmelidir.

---
