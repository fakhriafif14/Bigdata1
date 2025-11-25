# Modul Praktikum 5: Analisis Statistik Deskriptif dengan PySpark

## 1. Tujuan Praktikum

Setelah menyelesaikan modul ini, praktikan diharapkan mampu: 1.
Menginisialisasi session PySpark di Google Colab. 2. Memuat (load)
dataset ke dalam Spark DataFrame. 3. Menghitung statistik deskriptif
dasar (Mean, StdDev, Min, Max) menggunakan fungsi `.describe()`. 4.Menghitung statistik spesifik (Mean, Median, Modus, Varians, Skewness)
menggunakan `pyspark.sql.functions`. 5. Memahami tantangan dan solusi
dalam menghitung median (aproksimasi vs. eksak). 6. Melakukan
visualisasi distribusi data (Histogram & Box Plot) dari Spark DataFrame.

## 2. Persiapan Lingkungan (Google Colab)

### Instal PySpark

``` python
!pip install pyspark findspark -q
```

### Inisialisasi SparkSession

``` python
import findspark
findspark.init()

from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .master("local[*]") \
    .appName("PraktikumStatistikDeskriptif") \
    .getOrCreate()

print(spark)
```

## 3. Memuat dan Eksplorasi Data

### Muat Dataset ke Pandas

``` python
import seaborn as sns
pandas_df = sns.load_dataset('diamonds')
print(f"Jumlah baris di Pandas DF: {len(pandas_df)}")
pandas_df.head()
```

### Konversi ke Spark DataFrame

``` python
df = spark.createDataFrame(pandas_df)
df.show(5)
df.printSchema()
```

## 4. Analisis Statistik Deskriptif

### Cara Mudah: `.describe()`

``` python
df_described = df.describe()
df_described.show()
```

### Statistik Spesifik

``` python
from pyspark.sql.functions import mean, stddev, variance, min, max, col

price_stats = df.select(
    mean(col("price")).alias("mean_price"),
    stddev(col("price")).alias("stddev_price"),
    variance(col("price")).alias("variance_price"),
    min(col("price")).alias("min_price"),
    max(col("price")).alias("max_price")
)

price_stats.show()
```

### Modus (Mode)

``` python
from pyspark.sql.functions import desc

mode_cut = df.groupBy("cut") \
    .count() \
    .orderBy(desc("count"))

mode_cut.show(1)
```

### Median (Aproksimasi vs Eksak)

``` python
median_approx = df.summary("50%")
median_approx.show()

median_quantile = df.approxQuantile("price", [0.5], 0.01)
median_exact = df.approxQuantile("price", [0.5], 0.0)

print(median_quantile, median_exact)
```

## 5. Analisis Distribusi Data

### Sampling Data

``` python
sampled_df = df.sample(False, 0.1, seed=42)
```

### Konversi ke Pandas

``` python
viz_pandas_df = sampled_df.toPandas()
```

### Histogram Distribusi Harga

``` python
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(10, 6))
sns.histplot(viz_pandas_df['price'], kde=True, bins=50)
plt.title('Distribusi Harga Berlian (Sampel 10%)')
plt.show()
```

### Skewness

``` python
from pyspark.sql.functions import skewness
df.select(skewness("price")).show()
```

### Box Plot

``` python
plt.figure(figsize=(12, 7))
sns.boxplot(data=viz_pandas_df, x='cut', y='price',
            order=['Fair', 'Good', 'Very Good', 'Premium', 'Ideal'])
plt.title('Distribusi Harga berdasarkan Cut')
plt.show()
```

## 6. Latihan

1.  Hitung Mean, Median (aproksimasi), dan StdDev untuk kolom `carat`.
2.  Bandingkan rata-rata `price` untuk berlian `color = 'D'` vs
    `color = 'J'`. Manakah yang lebih mahal?
3.  Buat histogram untuk kolom `depth` menggunakan sampling 10%.
    Tentukan apakah distribusinya Normal, Skewed, atau Bimodal.
    

---

# 📘 Praktikum PySpark – Analisis Dataset Diamonds

## ⚙️ **Langkah-Langkah Praktikum**

### **1. Setup PySpark di Google Colab**

```bash
!pip install pyspark
```

```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("Praktikum PySpark - Diamond Analysis").getOrCreate()
```

---

### **2. Memuat Dataset**

Dataset diambil dari seaborn lalu diubah menjadi Spark DataFrame:

```python
# Download dataset diamonds.csv dari seaborn
import pandas as pd
import seaborn as sns

diamonds = sns.load_dataset('diamonds')
diamonds.to_csv('diamonds.csv', index=False)

# Membaca ke Spark DataFrame
df = spark.read.csv('diamonds.csv', header=True, inferSchema=True)

# Cek struktur DataFrame
df.printSchema()
df.show(5)
```

---

## 📊 **3. Latihan dan Penyelesaian**

### **3.1 Statistik Deskriptif Kolom `carat`**

#### **Mean & Standard Deviation**

```python
from pyspark.sql import functions as F

statistik = df.select(
    F.mean('carat').alias('Mean'),
    F.stddev('carat').alias('StdDev')
)
statistik.show()

```

#### **Median (Aproksimasi)**

Spark tidak memiliki median eksak untuk dataset besar → digunakan `approxQuantile()`.

```python
median_approx = df.approxQuantile("carat", [0.5], 0.01)[0]
print("Median (aproksimasi):", median_approx)

```

---

### **3.2 Perbandingan Rata-Rata `price` berdasarkan `color` (D vs J)**

```python
mean_price = df.groupBy('color').agg(F.mean('price').alias('avg_price'))
mean_price.show()

```
Lalu kamu bisa melihat mana yang lebih mahal:
```python
mean_price.orderBy('avg_price', ascending=False).show()
```
> Hasil umumnya menunjukkan bahwa **berlian color = 'D' lebih mahal** daripada color = 'J'.

---

### **3.3 Histogram Kolom `depth` (Menggunakan Sampling)**

Spark tidak cocok untuk visualisasi langsung → gunakan teknik **sampling**, lalu ubah menjadi Pandas.

#### **Sampling Data**

```python
# Ambil 10% data secara acak
sample_df = df.sample(fraction=0.1, seed=42)
pandas_df = sample_df.toPandas()
```

#### **Histogram**

```python
import matplotlib.pyplot as plt

plt.hist(pandas_df['depth'], bins=30, edgecolor='black')
plt.title('Histogram of Depth')
plt.xlabel('Depth')
plt.ylabel('Frequency')
plt.show()

```

### (c) Analisis Distribusi

* Jika bentuknya **simetris** → Normal
* Jika miring ke kanan/kiri → Skewed
* Jika ada dua puncak → Bimodal

Biasanya kolom `depth` pada dataset **berlian agak miring (skewed)**.

---
Langkah Akhir (Opsional): Simpan Hasil

Kamu bisa menyimpan hasil mean, median, dll ke file CSV jika diminta:
```python
result = [(float(statistik.collect()[0]['Mean']),
           float(statistik.collect()[0]['StdDev']),
           float(median_approx))]

result_df = spark.createDataFrame(result, ['Mean', 'StdDev', 'Median'])
result_df.show()

result_df.toPandas().to_csv('hasil_statistik.csv', index=False)
```

##  **7. Kesimpulan Praktikum**

1. **PySpark** memungkinkan pengolahan data besar tanpa memuat seluruh dataset ke memori, berbeda dengan Pandas.
2. Statistik deskriptif seperti **mean** dan **stddev** dapat dihitung secara cepat menggunakan fungsi Spark.
3. Median tidak tersedia secara eksak, tetapi fungsi **approxQuantile()** menyediakan pendekatan yang efisien.
4. Rata-rata harga berlian menunjukkan bahwa **color D lebih mahal daripada color J**.
5. Sampling membantu memindahkan sebagian data ke Pandas untuk keperluan visualisasi.
6. Histogram pada kolom `depth` memperlihatkan distribusi yang tidak benar-benar normal, melainkan sedikit skewed.


--
