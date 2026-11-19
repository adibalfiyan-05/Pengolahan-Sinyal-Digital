| No | Aspek | Paper | Replikasi kita | Alasan |
|---|---|---|---|---|
| 1 | Dataset | GTZAN 1000 klip, format mp3 | Versi Kaggle (andradaolteanu), .wav; 999 klip valid (file rusak dibuang) | Paper tidak menyebut sumber unduhan; jazz.00054 tidak bisa dibaca |
| 2 | Segmentasi | 30 s -> 3 s, penanganan sisa tidak disebut | 10 segmen non-overlap; 9 klip < 30 s di-zero-pad, kelebihan dipotong | Agar tiap lagu tepat 10 segmen |
| 3 | Split | 8:1:1, level tidak disebut | Protokol A per-segmen (dugaan paper) & B per-lagu, stratified, seed 42 | Ambiguitas paper; B untuk mengukur dampak leakage |
| 4 | Parameter STFT | Sec 3.2: n_fft 1024/hop 512, shape (128,129); Sec 4: window 2048/hop 512, shape (640,128) | Dua konfigurasi diuji: mel_1024 & mel_2048, Hann, center=True -> (128,130) | Paper tidak konsisten; shape (640,128) tidak bisa diturunkan dari segmen 3 s |
| 5 | MFCC | 13–20 koefisien, parameter framing tidak disebut | 20 koefisien dari log-mel 128 band, n_fft 2048, hop 512 | Nilai pasti tidak disebut; 20 = default librosa |
| 6 | Skala amplitudo | Log/dB, referensi tidak disebut | power_to_db(ref=1.0) | Referensi tidak disebut |
| 7 | Arsitektur Bi-RNN | Teks: 1 Bi-layer 128 + 1 RNN 128; Tabel 1: 3 layer RNN | Mengikuti Tabel 1: 3 layer (128-128-64), masing-masing dibungkus Bidirectional | Teks & tabel bertentangan; pilih desain seragam untuk 4 varian |
| 8 | Pooling & reshape | Ukuran pooling, padding, cara CNN->RNN tidak disebut | MaxPool 2x2 padding 'same' tiap blok; sumbu waktu jadi barisan RNN (5 langkah) | Asumsi paling sederhana yang kompatibel dg input mel (128) & MFCC (20) |
| 9 | Hiperparameter training | Optimizer, batch size, loss, lebar dense tidak disebut | Adam lr 1e-3, batch 64, sparse categorical cross-entropy, dense 128 ReLU | Pilihan standar; lr 0.001 dari baris Tabel 1 yang salah label |
| 10 | Normalisasi input | Tidak disebut | z-score per bin frekuensi/koefisien, statistik dari train set tiap protokol | Skala MFCC sangat timpang (koef-0 dominan); cegah kebocoran statistik test |
| 11 | Pemilihan model | Tidak disebut | Bobot epoch dengan val_accuracy terbaik (restore_best_weights) | Praktik standar |
| 12 | Stabilitas training | Tidak dilaporkan (single run, tanpa seed) | CNN+LSTM mel_1024 Protokol A diulang 3 seed (42, 43, 44) + 1 run clipnorm 1.0 | Run seed 42 divergen; hasil gagal tetap dilaporkan, clipnorm hanya untuk diagnosis |
| 13 | Learning rate CNN+LSTM | lr 0.001 (nilai 'teroptimasi' dari kandidat {0.1, 0.01, 0.001, 0.0001}) | CNN+LSTM mel_1024 memakai lr 0.0001; model lain tetap 0.001 | lr 0.001 menyebabkan kolaps (dead ReLU / gradien meledak, lihat fig_diagnosa_lstm); 0.0001 masih dalam ruang pencarian paper |
