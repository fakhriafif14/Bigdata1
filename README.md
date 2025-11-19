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

## 7. Kesimpulan Praktikum

-   PySpark dapat digunakan untuk analisis statistik skala besar.
-   Median aproksimasi jauh lebih efisien dibanding median eksak pada
    dataset besar.
-   Visualisasi perlu menggunakan sampling agar efisien.
