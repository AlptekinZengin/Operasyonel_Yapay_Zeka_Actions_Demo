# Operasyonel Yapay Zeka: GitHub Actions + CML Demo

[![cml](https://github.com/AlptekinZengin/Operasyonel_Yapay_Zeka_Actions_Demo/actions/workflows/cml.yaml/badge.svg)](https://github.com/AlptekinZengin/Operasyonel_Yapay_Zeka_Actions_Demo/actions/workflows/cml.yaml)

Bu proje, bir makine öğrenmesi modelinin her kod değişikliğinde **otomatik olarak eğitilip değerlendirildiği** ve sonuçların GitHub üzerinde rapor olarak paylaşıldığı küçük bir MLOps (operasyonel yapay zeka) örneğidir. Amaç, model eğitimini GitHub Actions ve [CML (Continuous Machine Learning)](https://cml.dev) ile bir CI/CD hattına bağlamayı göstermektir.

## Veri Seti

`operasyonel/heart.csv` dosyası 918 hastaya ait kayıtlar içerir. Hedef değişken `HeartDisease` (kalp hastalığı var: 1, yok: 0) olup model bu değeri tahmin eder.

| Özellik | Açıklama |
|---|---|
| `Age` | Yaş |
| `Sex` | Cinsiyet (M/F) |
| `ChestPainType` | Göğüs ağrısı tipi (ATA, NAP, ASY, TA) |
| `RestingBP` | Dinlenme kan basıncı |
| `Cholesterol` | Serum kolesterol |
| `FastingBS` | Açlık kan şekeri (> 120 mg/dl ise 1) |
| `RestingECG` | Dinlenme EKG sonucu |
| `MaxHR` | Ulaşılan maksimum kalp atış hızı |
| `ExerciseAngina` | Egzersize bağlı anjina (Y/N) |
| `Oldpeak` | Egzersizle oluşan ST depresyonu |
| `ST_Slope` | ST segmenti eğimi |
| `HeartDisease` | Hedef: kalp hastalığı var mı? |

## Model

`operasyonel/kalphastaligi.py` betiği şunları yapar:

1. Veri setini okur ve kategorik sütunları `LabelEncoder` ile sayısal hale getirir.
2. Veriyi %80 eğitim, %20 test olarak böler (`random_state=13`).
3. `RandomForestClassifier(n_estimators=10, max_depth=2)` modelini eğitir.
4. Eğitim ve test doğruluğunu `metrikler.txt` dosyasına yazar.
5. Özellik önem grafiğini `ozellik_onem.png` olarak kaydeder.

Betik ekransız (headless) ortamlarda çalışması için matplotlib'in `Agg` arka ucunu kullanır. Çıktılar depo kök dizinine yazılır ve `.gitignore` ile sürüm kontrolüne alınmaz.

## CI/CML Hattı

`.github/workflows/cml.yaml` iş akışı, her `push` ve `pull_request` olayında çalışır:

1. `actions/checkout` ile kod alınır.
2. `actions/setup-python` ile Python 3.12 kurulur.
3. `iterative/setup-cml` ile CML aracı kurulur.
4. `requirements.txt` içindeki bağımlılıklar yüklenir.
5. `python operasyonel/kalphastaligi.py` ile model eğitilir.
6. Metrikler ve grafik `model-raporu` adıyla iş akışı çıktısı (artifact) olarak yüklenir; metrikler ve grafiğe bağlantı `rapor.md` dosyasında birleştirilip `cml comment create` ile ilgili commit'e (veya pull request'e) yorum olarak eklenir.

Yorum için ek bir anahtar gerekmez; iş akışı yerleşik `GITHUB_TOKEN` değerini kullanır (`contents: write`, `pull-requests: write` izinleri tanımlıdır).

## Yerel Çalıştırma

```bash
git clone https://github.com/AlptekinZengin/Operasyonel_Yapay_Zeka_Actions_Demo.git
cd Operasyonel_Yapay_Zeka_Actions_Demo

python -m venv .venv
# Windows: .venv\Scripts\activate   |   Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt

python operasyonel/kalphastaligi.py
```

Çalıştırma sonunda kök dizinde `metrikler.txt` ve `ozellik_onem.png` dosyaları oluşur.

## Klasör Yapısı

```
.
├── .github/workflows/cml.yaml   # GitHub Actions + CML iş akışı
├── operasyonel/
│   ├── heart.csv                # Kalp hastalığı veri seti
│   └── kalphastaligi.py         # Model eğitimi ve raporlama betiği
├── requirements.txt             # Python bağımlılıkları
├── .gitignore
├── LICENSE
└── README.md
```

## Lisans

Bu proje [Apache License 2.0](LICENSE) ile lisanslanmıştır.
