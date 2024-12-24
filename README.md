# Web-Scraping S&P500
### 23.83.0970
### 23TK01
### Andi Ahmad Fauzan
<hr>

## Import
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# requests digunakan untuk mengirim permintaan HTTP ke situs web
# BeautifulSoup digunakan untuk mem-parsing HTML dari sius web yang di-scrape
# pandas untuk memanipulasi data dan pengelolaan data dalam format tabel
# matplotlib.pyplot untuk membuat visualisasi data
# Library visualisasi data berbasis matplotlib dengan desain yang lebih menarik
```

## url
```python
url = "https://stockanalysis.com/list/sp-500-stocks/"

# Url sari halaman web yang berisi daftar saham S&P500 yang akan di-scrape
```

## Fungsi untuk melakukan scraping data
```python
def scrape_sp500_data(url):

# Mendefinisikan fungsi scrape_sp500_data untuk mengambil data dari URL yang diberikan.
```

#### Session dan Request
```python
    try:
       with requests.Session() as session:
          response = session.get(url)
          response.raise_for_status()
          soup = BeautifulSoup(response.text, 'html.parser')

# Membuka sesi HTTP dengan requests.Session.
# Mengambil konten halaman dengan session.get(url).
# Memastikan permintaan berhasil dengan response.raise_for_status().
# Mem-parsing HTML halaman menggunakan BeautifulSoup.
```

#### Mencari Tabel
```python
      table = soup.select_one('table')
      if not table:
          raise ValueError("Tabel data tidak ditemukan di halaman.")

# Mencari elemen tabel dalam HTML dengan select_one.
# Jika tabel tidak ditemukan, muncul pesan error.
```

#### Mengambil Header dan Data
```python
      headers = [th.text.strip() for th in table.select('thead th')]
      rows = table.select('tbody tr')
      data = [[col.text.strip() for col in row.select('td')] for row in rows]

# headers Mengambil teks dari elemen thead th (header tabel).
# rows Mengambil semua baris data (tr) dari tbody.
# data Menyusun setiap kolom dalam baris ke dalam list dua dimensi.
```

#### Membuat DataFrame
```python
        df = pd.DataFrame(data, columns=headers)
        df = df.rename(columns={
            'Symbol': 'Tickers',
            'Company Name': 'Company',
            'Market Cap': 'MCap',
            'Stock Price': 'Price',
            '% Change': '24h Change %',
        })
# Membuat DataFrame menggunakan pandas.
# Mengganti nama kolom untuk lebih mudah dipahami dan digunakan.
```

#### Filter dan Konversi Data
```python
        df = df[['Tickers', 'Company', 'MCap', 'Price', '24h Change %']].copy()
        df['Price'] = pd.to_numeric(df['Price'], errors='coerce')
        df['24h Change %'] = pd.to_numeric(df['24h Change %'].str.replace('%', ''), errors='coerce')
        df['MCap'] = df['MCap'].str.replace('[^\d]', '', regex=True).astype(float)

# Memilih kolom penting untuk analisis.
# Mengubah tipe data kolom Price dan 24h Change % menjadi numerik.
# Membersihkan kolom MCap dengan menghapus karakter non-numerik dan mengonversi menjadi tipe float.
```

#### Menambahkan rentang harga
```python
        bins = [0, 50, 100, 200, 500, 1000, float('inf')]
        labels = ['<50', '50-100', '100-200', '200-500', '500-1000', '>1000']
        df['Price Range'] = pd.cut(df['Price'], bins=bins, labels=labels)

# Menambahkan kategori rentang harga berdasarkan nilai Price.
# bins Batas rentang harga.
# labels Label untuk setiap rentang.
```

## Visualisasi data menggunakan Bar Plot
```python
def visualize_data(df):
    # Bar plot untuk perusahaan top berdasarkan kapitalisasi pasar
    top_companies = df.nlargest(10, 'MCap')
    plt.figure(figsize=(14, 6))
    sns.barplot(x='Company', y='MCap', data=top_companies, palette='viridis')
    plt.xticks(rotation=45, ha='right')
    plt.title("Top 10 Companies by Market Cap")
    plt.ylabel("Market Cap (in trillions)")
    plt.show()

# Fungsi untuk visualisasi data:
# nlargest Memilih 10 perusahaan dengan kapitalisasi pasar terbesar.
# Membuat bar plot menggunakan seaborn.barplot.
# Menambahkan judul dan memformat sumbu.

```


## Melakukan perintah print untuk scraping & visualisasi data
```python
sp500_data = scrape_sp500_data(url)
if sp500_data is not None:
    print("Scraped Data (Sample):")
    print(sp500_data.head())

# Memanggil fungsi scrape_sp500_data dan menampilkan contoh data jika berhasil.
```

## Save ke CSV
```python
    csv_file = "S&P500.csv"
    sp500_data.to_csv(csv_file, index=False)
    print(f"Data saved to {csv_file}")

#Menyimpan data ke file CSV dengan nama S&P500.csv.
```

## Error handling
```python
else:
    print("Gagal mendapatkan data.")


# Menampilkan pesan jika data gagal diambil.
```

