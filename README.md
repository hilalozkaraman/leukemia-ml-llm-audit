# ALL/AML Lösemi Mikrodizi Verilerinde Keşifsel Analiz ve Platformlar Arası Karşılaştırma

**Proje durumu:** Dönem projesi — **Ara Rapor / EDA aşaması**. Bu sürümde henüz ALL/AML tahmini yapan makine öğrenmesi modellerinin nihai performans karşılaştırması veya LLM deney sonuçları bulunmamaktadır.

## Projenin amacı

Bu proje, gen ifadesi (mikrodizi) verilerinden **akut lenfoblastik lösemi (ALL)** ve **akut miyeloid lösemi (AML)** alt tiplerinin ayrımını incelemek amacıyla hazırlanmıştır. Ara rapor kapsamında, yüksek boyutlu gen ifade verilerinin kalitesi ve sınıf dağılımları araştırılmakta; ayırt edici genler belirlenmekte; farklı mikrodizi platformlarında elde edilen gen profillerinin benzerliği değerlendirilmektedir.

Başlıca araştırma soruları:

- **RQ1:** Az sayıda genden oluşan paneller, ALL/AML ayrımını ne ölçüde destekleyebilir ve seçilen genler farklı veri bölmelerinde ne kadar kararlı kalır?
- **RQ2:** Golub ve GSE13159 veri setleri arasında ortak genler ve ayırt edicilik örüntüleri ne kadar tutarlıdır? Seçilen panellerin platformlar arasında taşınabilirliği final aşamasında nasıl değerlendirilebilir?
- **RQ3 (planlanan bonus):** Gen adlarına ilişkin ön bilginin LLM destekli analizlere etkisi nedir? Gerçek, anonimleştirilmiş ve karıştırılmış gen adlarıyla karşılaştırma yapılabilir mi?

> **Kapsam notu:** Ara rapordaki AUC analizi, bir örneğin **hangi veri setinden geldiğini** ayırt etmeye yöneliktir; ALL/AML teşhis başarısı değildir. Klinik kullanım veya tanı amacı taşımaz.

## Veri setleri

| Özellik | Golub ALL/AML | GSE13159 (MILE, ALL/AML alt kümesi) |
|---|---|---|
| Kaynak | Golub ve ark., 1999; derste kullanılan Kaggle kopyası | [NCBI GEO — GSE13159](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE13159) |
| Platform | Affymetrix Hu6800 / GPL80 | Affymetrix HG-U133 Plus 2.0 / GPL570 |
| Örnek sayısı | 72 (38 eğitim, 34 bağımsız test) | 1.292 filtrelenmiş akut lösemi örneği (toplam seride 2.096) |
| Prob sayısı | 7.129 | 54.675 |
| ALL / AML | 47 / 25 | 750 / 542 |
| EDA ifade ölçeği | Eşik kırpma sonrası log2 | GEO'dan sağlanan DQN3 normalize sinyal (0–1) |

**Önemli:** ALL ve AML'nin ikisi de lösemi alt tipidir; ALL “kanser olmayan” bir sınıf değildir. Golub örnekleri ve etiketleri CSV sütun konumuna göre değil, hasta kimliğine göre eşleştirilir.

Veri kaynakları ve anotasyonlar: [GPL80](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL80), [GPL570](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL570), [GSE13159 Series Matrix dizini](https://ftp.ncbi.nlm.nih.gov/geo/series/GSE13nnn/GSE13159/matrix/). Veri dosyalarının yeniden paylaşım ve kullanım koşulları ilgili kaynaklardan kontrol edilmelidir.

## Bu sürümde yapılan analizler

1. **Golub veri kontrolü:** Hasta kimliği–etiket eşleştirmesi; `Present / Absent / Marginal (P/A/M)` çağrıları; eksik, sonsuz, negatif ve uç ifade değerleri.
2. **Golub EDA:** Sınıf dağılımı, ham ve log2 ifade dağılımları, PCA, t-SNE, Welch t-testi, Benjamini–Hochberg FDR, ilk 30 prob için hiyerarşik kümeli ısı haritası ve eğitim/test farkları.
3. **GSE13159 EDA:** ALL/AML alt tip filtresi, DQN3 sinyal kalitesi, sınıf dağılımları, PCA/t-SNE, Welch t-testi, FDR ve ısı haritası.
4. **Veri setleri arası karşılaştırma:** GPL80/GPL570 ile prob–gen sembolü eşlemesi, ortak genler, t-istatistiği korelasyonu, en yüksek ayırt ediciliğe sahip genlerde örtüşme, veri seti kimliği ROC-AUC ve MILE–MILE negatif kontrolü.

Aynı gen sembolüyle eşleşen birden fazla prob varsa **alfabetik sıradaki ilk prob** seçilir; ölçüm değerlerine göre en iyi prob seçimi yapılmaz. Veri seti kimliği AUC hesabında imputasyon, ölçekleme ve PCA çapraz doğrulama katının içinde uygulanır. Buna karşılık bazı **EDA görselleştirmeleri betimsel amaçlıdır** ve final modellemesinin doğrulama prosedürü olarak kullanılmamalıdır.

## Çalıştırma — Google Colab

**Gereksinimler:** Google Colab, erişilebilir bir Google Drive hesabı, Python ve `numpy`, `pandas`, `scipy`, `scikit-learn`, `matplotlib`, `seaborn` paketleri. Kullanılan rastgelelik tohumu: **42**. Yerel ortam için `requirements.txt` sağlanmıştır; Google Colab bu paketlerin çoğunu önceden içerir.

1. Bu depodaki notebook'u Google Colab ile açın: `leukemia_ml_llm_audit.ipynb` (**depoda farklı bir ad kullanıyorsanız burada güncelleyin**).
2. Google Drive içinde aşağıdaki dosyaları bulundurun:
   - `archive.zip` — Golub CSV dosyalarını içeren arşiv (`data_set_ALL_AML_train.csv`, `data_set_ALL_AML_independent.csv`, `actual.csv`).
   - `GSE13159_series_matrix.txt.gz` — GEO Series Matrix.
   - `GPL570.annot.gz` — GSE13159 platform anotasyonu.
   - `GPL80.annot.gz` — Golub platform anotasyonu; notebook, dosya yoksa NCBI'den indirmeyi dener.
3. Notebook'un **ADIM 0: Google Drive bağlantısı ve dosya yolları** hücresindeki `GOLUB_ZIP_PATH`, `GSE13159_GZ_PATH`, `GPL570_PATH`, `GPL80_PATH` değerlerini kendi Drive yapınıza göre düzenleyin.
4. Hücreleri sırasıyla **ADIM 0 → ADIM 1 → ADIM 2 → ADIM 3 → ADIM 4** çalıştırın. Ardından `save_results()` fonksiyonunu çalıştırın. `save_results()` hücresinde eski `if __name__ == "__main__"` bloğu duruyorsa analizlerin tekrar çalışabileceğine dikkat edin.
5. Çıktıları `./eda_outputs/` klasöründen alın. Gerekirse klasörü Drive'a kopyalayın.

```text
repo/
├── leukemia_ml_llm_audit.ipynb      # EDA ve veri setleri arası analiz
├── README.md
├── requirements.txt
└── eda_outputs/                    # Çalıştırma ile oluşturulur
    ├── results.json
    ├── environment_versions.txt
    ├── fig1_golub_distributions.png
    ├── fig2_golub.png
    ├── fig3_golub_top30_heatmap.png
    ├── fig4_gse13159_subtypes_distributions.png
    ├── fig5_gse13159.png
    ├── fig6_gse13159_top30_heatmap.png
    ├── fig7_cross_dataset_concordance.png
    └── tables/
        ├── golub_quality.csv
        ├── golub_train_de_top100.csv
        ├── gse13159_quality.csv
        ├── gse13159_de_top100.csv
        ├── mile_class_mapping_check.csv
        └── common_genes.csv
```

Görseller PNG (300 dpi) ve PDF formatında üretilir. `results.json` rapordaki sayısal bulgular için temel kayıt dosyasıdır. `environment_versions.txt` çalışma sırasında yüklü kütüphane sürümlerini kaydeder.

## Mevcut EDA çıktılarından özet (10 Kasım 2026)

- Golub: 72 örnek; 7.129 prob; eğitim/test sınıf sayıları rehberle uyumlu.
- GSE13159: 1.292 ALL/AML örneği; 54.675 prob; değer aralığı **0–1 (DQN3, log2 değil)**.
- Golub prob–gen eşleme kapsamı: **%84,8**; iki veri setinde **5.207 ortak gen**.
- Ortak genlerde Welch t-istatistiklerinin Spearman korelasyonu: **ρ = 0,4036**.
- En ayırt edici 100 gen arasındaki örtüşme: **27 gen**; seçilen Golub top-100 genlerinin işaret uyumu: **%90**.
- Dengeli Golub–MILE veri seti kimliği sınıflandırmasında ROC-AUC: **1,000**; örnek bazlı gen sıralaması sonrasında da **1,000**. MILE–MILE negatif kontrolü: **0,523**.

Bu AUC değerleri **ALL/AML sınıflandırıcı başarısı değildir** ve yalnızca “platform etkisi” şeklinde yorumlanamaz; platform, kohort, prob ve normalizasyon farkları iç içedir. Özet sayılar, bu sürümün paylaşılan çalıştırma kayıtlarını yansıtır; yeniden çalıştırmada `results.json` esas alınmalıdır.

## Bilimsel sınırlılıklar ve gelecek çalışma

- Golub'un örnek sayısı küçük, gen sayısı yüksektir (`p ≫ n`).
- Golub ve MILE, farklı mikrodizi platformları ve veri ön işleme yöntemleri kullanır. Ortak **gen sembolü**, probların birebir biyolojik eşdeğerliği anlamına gelmez.
- Gen paneli taşınabilirliği ve ALL/AML model performansı henüz değerlendirilmemiştir. Final aşamasında **lojistik regresyon (L1/Elastic Net), SVM ve Random Forest** ile iç içe, tabakalı çapraz doğrulama; bağımsız test ve panel kararlılığı analizleri planlanmaktadır.
- LLM ile gen adlarını anonimleştirme/karıştırma karşılaştırması **planlanan RQ3 / bonus** kapsamındadır; bu depoda tamamlanmış LLM sonuçları bulunmaz.
- Keşifsel analizde sınıf etiketleriyle oluşturulan görseller model seçimi için doğrudan kullanılmamalıdır. Nihai model seçimi ve öznitelik seçimi eğitim katları içinde gerçekleştirilecektir.

## GenAI kullanımı

Projenin literatür düzenlemesi, kod incelemesi ve metodolojik iyileştirmelerinde üretken yapay zekâ araçlarından destek alınmıştır. Kod, istatistiksel tercihler ve bilimsel yorumların doğrulanması proje sahibinin sorumluluğundadır. **Kullanılan araç/sürüm, katkı, insan tarafından yapılan değişiklik ve kontrol adımları** ara raporun GenAI beyanında ayrıca belgelenecektir. Bu bölüm, gerçekleşen işlemlere göre güncellenmelidir.

## Temel kaynaklar

1. T. R. Golub ve ark., “Molecular classification of cancer: Class discovery and class prediction by gene expression monitoring,” *Science*, 1999. DOI: [10.1126/science.286.5439.531](https://doi.org/10.1126/science.286.5439.531).
2. T. Haferlach ve ark., “Clinical utility of microarray-based gene expression profiling in the diagnosis and subclassification of leukemia,” *Journal of Clinical Oncology*, 2010. DOI: [10.1200/JCO.2009.23.4732](https://doi.org/10.1200/JCO.2009.23.4732).
3. [NCBI GEO GSE13159](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE13159) ve platform anotasyonları [GPL80](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL80) / [GPL570](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GPL570).

