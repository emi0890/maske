# maske 🇹🇷 — Türkçe KVKK PII Maskeleme NER

[🇬🇧 English](README_en.md) · [🇹🇷 Türkçe](README.md)

[![Hugging Face · maske](https://img.shields.io/badge/%F0%9F%A4%97%20maske-emi08%2Fmaske-yellow)](https://huggingface.co/emi08/maske)
[![Hugging Face · bakeoff](https://img.shields.io/badge/%F0%9F%A4%97%20bakeoff-emi08%2Fmaske--modernbert--tr--bakeoff-yellow)](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff)
[![Hugging Face · baseline](https://img.shields.io/badge/%F0%9F%A4%97%20baseline-emi08%2Fmaske--berturk--baseline-yellow)](https://huggingface.co/emi08/maske-berturk-baseline)

## Genel Bakış

`maske`, Türkçe metinlerdeki kişisel verileri **6698 sayılı Kişisel Verilerin Korunması Kanunu (KVKK)** kapsamında tespit etmek ve otomatik maskelemek için geliştirilmiş bir token sınıflandırma (NER) model ailesidir. Aile; sentetik veriyle eğitilmiş ana model `emi08/maske` ile birlikte, mimari seçimini ölçmek için eğitilen `emi08/maske-modernbert-tr-bakeoff` ve `emi08/maske-berturk-baseline` referans modellerinden oluşur. Tüm eğitim verisi sentetik olarak üretilir; bu sayede eğitimde hiçbir gerçek kişisel veri kullanılmaz. Modeller dokuz varlık tipini BIO şemasıyla etiketler ve span-exact (tam aralık) ölçütüyle değerlendirilir.

## Modeller

| Model | Not | WikiANN-tr F1 |
|---|---|---|
| [emi08/maske](https://huggingface.co/emi08/maske) | Ana model — sentetik KVKK verisiyle eğitildi; e-posta tespitini iyileştiren eğitim güncellemesi sürüyor | **0.9317** |
| [emi08/maske-modernbert-tr-bakeoff](https://huggingface.co/emi08/maske-modernbert-tr-bakeoff) | Mimari karşılaştırması — modernbert-tr-base tabanı | 0.929 |
| [emi08/maske-berturk-baseline](https://huggingface.co/emi08/maske-berturk-baseline) | Aşama 1 referans modeli — BERTurk tabanı | 0.929 |

## Varlık Tipleri

| Tip | Açıklama | Örnek |
|---|---|---|
| PER | Kişi adı | Ayşe Yılmaz |
| ORG | Kurum / şirket | Kolaykargo A.Ş. |
| LOC | Şehir / ülke | Kadıköy, Ankara |
| ADDR | Açık adres | Bağdat Cad. No:12 D:3, Kadıköy/İstanbul |
| TC_NO | T.C. Kimlik No (11 hane, kontrol basamaklı) | 34628194872 |
| IBAN | TR IBAN (mod-97 doğrulamalı) | TR33 0006 1005 1978 6457 8413 26 |
| PHONE | Telefon numarası | 0532 123 45 67, +90 532 123 45 67 |
| EMAIL | E-posta adresi | ad.soyad@firma.com.tr |
| PLATE | Araç plakası | 34 ABC 123 |

## Etiket Kümesi (19)

~~~text
O
B-PER   I-PER
B-ORG   I-ORG
B-LOC   I-LOC
B-ADDR  I-ADDR
B-TC_NO I-TC_NO
B-IBAN  I-IBAN
B-PHONE I-PHONE
B-EMAIL I-EMAIL
B-PLATE I-PLATE
~~~

## Hızlı Başlangıç

~~~python
from transformers import pipeline

pipe = pipeline(
    "token-classification",
    model="emi08/maske",
    aggregation_strategy="simple",
)

text = (
    "Sayın Ayşe Yılmaz, TC kimlik no 12345678901. "
    "Ödemeyi TR33 0006 1005 1978 6457 8413 26 IBAN hesabına yapın. "
    "Sorularınız için ayse.yilmaz@ornek.com.tr adresine yazın."
)

for entity in pipe(text):
    print(f"{entity['entity_group']:8s} {entity['word']}  ({entity['start']}–{entity['end']})")
~~~

Metni otomatik maskelemek için:

~~~python
def mask_text(text: str, pipe) -> str:
    """Tespit edilen PII varlıklarını [TIPLERİYLE] değiştirir."""
    entities = sorted(pipe(text), key=lambda e: e["start"], reverse=True)
    for e in entities:
        text = text[: e["start"]] + f"[{e['entity_group']}]" + text[e["end"] :]
    return text

print(mask_text("Ayşe Yılmaz adına fatura, 34 ABC 123 plakalı araca kesildi.", pipe))
# [PER] adına fatura, [PLATE] plakalı araca kesildi.
~~~

## Değerlendirme

Tüm sayılar aynı span-exact değerlendirme koduyla üretilmiştir: bir varlığın doğru sayılması için hem tipin hem de tam karakter aralığının eşleşmesi gerekir.

### WikiANN-tr karşılaştırması

| Model | P | R | F1 |
|---|---|---|---|
| emi08/maske | 0.9260 | 0.9374 | **0.9317** |
| emi08/maske-berturk-baseline | 0.923 | 0.934 | 0.929 |
| emi08/maske-modernbert-tr-bakeoff | 0.923 | 0.935 | 0.929 |
| savasy/bert-base-turkish-ner-cased | 0.919 | 0.930 | 0.925 |
| Werea-co/Werea-TR-NER | 0.907 | 0.917 | 0.912 |
| akdeniz27/bert-base-turkish-cased-ner | 0.699 | 0.649 | 0.673 |

### emi08/maske — küme bazlı sonuçlar

| Küme | P | R | F1 |
|---|---|---|---|
| Sentetik test | — | — | 1.0000 |
| WikiANN-tr | 0.9260 | 0.9374 | 0.9317 |
| Werea-KVKK-Bench (240 satır) | 0.1789 | 0.4875 | 0.2617 |
| Gerçek metin (29 belge) | 0.6809 | 0.9143 | 0.7805 |

### Gerçek metin değerlendirme kümesi

Elle derlenen küme **29 belge / 35 gold varlık** içerir; belgelerin 8'i tuzak dokümandır (kontrol basamağı hatalı TC kimlik numaraları, geçersiz IBAN'lar, 11 haneli sipariş numaraları, kart numaraları, VKN ve URL'ler gibi). Tuzak değerler gold'a dahil değildir ve model bunları raporlarsa yanlış pozitif sayılır; mevcut sürüm bu belgelerde 7 FP üretmiştir. Tür bazında gold dağılımı:

~~~text
PER 7 · ORG 5 · LOC 5 · TC_NO 5 · PHONE 4 · ADDR 3 · IBAN 2 · EMAIL 2 · PLATE 2 — toplam 35
~~~

**Bilinen sorun (güncel eğitimde gideriliyor):** gerçek metinlerde e-posta adresleri parçalı yakalanmaktadır (EMAIL F1 = 0.0). Model kartındaki kurtarma son işlemi — '@' içermeyen EMAIL tahminlerinin atılması ve geçerli e-posta regex'iyle çakışan tahminlerin tam adres aralığına normalize edilmesi — bu durumu üretimde giderir.

## Durum ve Yol Haritası

- [x] Ana model `emi08/maske` + baseline / bakeoff karşılaştırması ve harici model kıyaslaması
- [ ] **Eğitim güncellemesi (sürüyor):** e-posta tespiti için booster verisi (tam span pozitifleri + gözlemlenen yanlış pozitifler için zorlayıcı negatifler) ve kurtarma son işlem katmanı
- [ ] Güncelleme bitince eklenecek: gerçek metin toplam F1, gerçek metin EMAIL F1, Werea-KVKK-Bench F1 ve tür bazlı gerçek metin tablosu
- [ ] Model kartlarının nihai güncellenmesi

## Lisans

Bu proje **Apache 2.0** lisansıyla sunulmaktadır.

## Atıf

~~~bibtex
@misc{maske_2026,
  author       = {emi08},
  title        = {maske: Turkish Personal Data (PII) Detection for KVKK Workflows},
  year         = {2026},
  url          = {https://huggingface.co/emi08/maske}
}
~~~
