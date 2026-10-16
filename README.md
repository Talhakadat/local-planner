# Autonomous Vehicle Local Planner

## Problem

Aracın, çevresindeki engellere ve hedef konumuna göre en iyi yerel rotayı üretmesi hedeflenmektedir. Yerel planlayıcı (DWA / MPC), çevredeki dinamik engellerin gelecekteki hareketlerini tahmin eden 3 farklı yapılandırma ile karşılaştırılacaktır:

| Yapılandırma | Tahmin Modeli | Açıklama |
|---|---|---|
| **Base** | Sabit hız modeli (constant velocity) | Öğrenme yok, yalnızca matematiksel modelleme |
| **LSTM** | LSTM + CVAE | Geçmiş yörüngelerden multi-modal tahmin |
| **Transformer** | Transformer + CVAE | Attention ile ajan etkileşimlerini modelleyen multi-modal tahmin |

Sinir ağları doğrudan aracı sürmez; diğer ajanların gelecekteki yörüngelerini tahmin eder ve yerel planlayıcı bu tahminlere göre çarpışmasız yörünge üretir.

## Goals
-   ROS2 entegrasyonu
-   SLAM ile konumlandırma ve haritalama
-   Nesne tespit algoritması
-   Nesne takip algoritması
-   Gazebo'da simülasyon
-   Güvenlik kontrolü (safety monitor)
-   Multi-modal yörünge tahmini (CVAE)
-   Autoware'in incelenmesi ve kendi çözümlerimle karşılaştırılması
-   Gerçek araç üzerinde kontrol
-   3 yapılandırmanın benchmark testi

## Non-Goals
-   CARLA simülasyonu

## Teknik Yığın

> Netleştirilecek (bkz. [Açık Sorular](#açık-sorular))

-   ROS2: _Humble / Jazzy_
-   Simülasyon: _Gazebo (Harmonic veya güncel sürüm)_
-   Diller: C++ (perception, planning, control), Python (ML)
-   ML: PyTorch, gerçek zamanlı çıkarım için ONNX / TensorRT
-   Araç modeli: _Ackermann / diferansiyel sürüş_
-   Hedef donanım: _belirlenecek_

## Sistem Mimarisi

```
                    ┌───────────────┐
                    │    Sensors    │
                    │ LiDAR, Kamera │
                    └───────┬───────┘
                ┌───────────┴────────────┐
                ↓                        ↓
        ┌───────────────┐        ┌───────────────┐
        │  Perception   │        │ Localization  │
        │  (Detection)  │        │    (SLAM)     │
        └───────┬───────┘        └───────┬───────┘
                ↓                        │
        ┌───────────────┐                │
        │   Tracking    │                │
        └───────┬───────┘                │
                ↓                        │
     ┌─────────────────────┐             │
     │     Prediction      │             │
     │ Base / LSTM / Trans │             │
     │      (+ CVAE)       │             │
     └──────────┬──────────┘             │
                ↓                        ↓
        ┌───────────────┐        ┌───────────────┐
        │ Local Planner │←───────│ Global Route  │
        │  (DWA / MPC)  │        │  (waypoints)  │
        └───────┬───────┘        └───────────────┘
                ↓            ↑ occupancy grid (Perception'dan)
        ┌───────────────┐
        │Safety Monitor │
        └───────┬───────┘
                ↓
        ┌───────────────┐
        │  Controller   │
        └───────────────┘
```

### Sensors & Perception

#### LiDAR -> point cloud pipeline

```
LiDAR
↓
PointCloud2
↓
Voxel Filter
↓
RANSAC Ground Removal
↓
Euclidean Clustering
↓
3D Bounding Boxes ──→ Tracking
↓
Local Occupancy Grid
↓
Collision Checking
↓
Local Planner
↓
cmd_vel / ackermann_cmd
```

-   Bu kavramları önce kendim öğrenip uygulayacağım, sonrasında Autoware GitHub sayfasından bu problemlerin nasıl çözüldüğünü inceleyeceğim.

#### RGB Kamera

-   Görüntü üzerinden engeller tespit edilecek (2B bounding box + sınıf).
-   Tek kameradan derinlik bilgisi elde edilemediği için 2B tespitler LiDAR nokta bulutuna izdüşürülerek engellerin 3B konumları çıkarılacak (camera–LiDAR fusion).
-   Bunun için kamera–LiDAR kalibrasyonu (intrinsic + extrinsic) yapılacak.

### Localization (SLAM)
-   Aracın harita üzerindeki konumu SLAM ile belirlenecek; planlayıcı ve tahmin modülü bu konumu kullanacak.

### Tracking
-   Tespit edilen nesnelere ID atanarak zaman içinde takip edilecek (Kalman Filter / SORT tabanlı).
-   Tahmin modelleri, her nesnenin geçmiş yörüngesini bu modülden alacak.

### Prediction

#### Base
-   Sabit hız modeli ile engellerin gelecekteki konumları tahmin edilecek. Referans (baseline) olarak kullanılacak.

#### LSTM
-   Takip edilen nesnelerin geçmiş yörüngeleri LSTM ile kodlanarak multi-modal yörünge tahmini yapılacak.

#### Transformer
-   Elde edilen bilgiler encode edilip attention mekanizması ile birleştirilecek; ajanlar arası etkileşim modellenecek.

#### CVAE
-   Conditional Variational Autoencoder, LSTM ve Transformer modellerinin çıkış katmanında kullanılarak birden fazla olası yörünge (multi-modal) üretilecek.

### Local Planner
-   DWA ve MPC yerel planlayıcı olarak kullanılacak.
-   Prediction modülünden tahmin geldiyse planlama bu tahminler eşliğinde yapılacak; aksi takdirde planlayıcı kendi matematiksel modeli ile (Base) planlama yapacak.
-   Hedef, Global Route modülünden gelen ara noktalar (waypoints) ile verilecek.

### Safety Monitor
-   Time-to-collision (TTC) kontrolü
-   Acil durdurma (emergency stop)
-   Watchdog: modüllerden veri gelmezse veya gecikirse aracı güvenli duruma alma
-   Fallback: öğrenilmiş model hata verirse veya tahmin güvenilmezse Base yapılandırmasına geçiş

### Controller
-   Planlanan yörüngenin takibi (Pure Pursuit / Stanley / PID).

## Veri ve Eğitim
-   Gazebo senaryolarından rosbag ile veri toplanması
-   Hazır veri setleri ile ön eğitim (ETH/UCY, nuScenes, Argoverse 2 — seçilecek)
-   Sim-to-real farkının gerçek araç testlerinde değerlendirilmesi
-   Eğitilen modellerin ONNX / TensorRT'ye aktarılması

## Benchmark

### Tahmin Metrikleri
-   ADE / FDE
-   minADE_k / minFDE_k (multi-modal)
-   Miss rate

### Planlama Metrikleri
-   Başarı oranı (hedefe ulaşma)
-   Çarpışma oranı
-   Engellere minimum mesafe
-   Hedefe varış süresi ve yol uzunluğu
-   Jerk (sürüş konforu)

### Sistem Metrikleri
-   Gecikme (ms) ve çalışma frekansı (Hz)

### Senaryolar
-   Statik engeller
-   Dinamik engeller (yaya / araç)
-   Dar geçit
-   Kalabalık ortam

Tüm senaryolar sabit seed ile tekrarlanabilir şekilde çalıştırılacak.

## Yol Haritası

| Faz | İçerik |
|---|---|
| 1 — MVP |ROS2, GAZEBO ortamları kurulup, simülasyona hazır hale getirilecekler. |
| 2 | Lidar ve Kamera sensörleri kurulacak; point cloud konusu çalışılacak ve son haline getirilecek. |
| 3 | Kamera üstünden obje tespit, takip ve ID atama işlemleri yapılacak | 
| 4 | lidar ve kamera füzyonu yapılacak. | 
| 5 | SLAM ve Base model uygulaması ve testleri. |
| 6 | LSTM ve CVAE konuları çalışılıp projeye entegre edilecek. | 
| 7 | Transformers ve CVAE projeye entegre edilip testler yapılacak. |
| 8 | Arayüz geliştirilecek. | 
| 9 | Gerçek araç testleri. |


## Açık Sorular
-   ROS2 dağıtımı ve Gazebo sürümü?
-   Araç modeli: Ackermann mı, diferansiyel sürüş mü?
-   Base planner için DWA mı, MPC mi, yoksa ikisi de mi karşılaştırılacak?
-   Autoware yalnızca referans olarak mı incelenecek, yoksa bazı modülleri doğrudan kullanılacak mı?
-   Gerçek araç için hedef donanım (ör. Jetson)?

## License

MIT — bkz. [LICENSE](LICENSE)
