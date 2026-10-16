# Autonomous Vehicle Local Planner

## Problem 

Araç 3 farklı yerel planlayıcı algoritması kullanarak(Base, LSTM ve Transformers) çevresindeki engellere ve hedef konumuna göre en iyi rotaları üretmesi hedeflenmektedir. 

## Goals
-   ROS2 entegrasyonu
-   SLAM konumlandırma ve haritalama
-   Nesne tespit algoritması
-   Nesne takip algoritması
-   Gazeboda simülasyon
-   Güvenlik kontrolü
-   Multi model learning
-   Autoware kullanımı ve algoritma incelemesi
-   Gerçek araba üstünde kontrol
-   3 algoritmanın da benchmark testi

## Non-Goals
-   CARLA simülasyon

## Sistem Mimarisi

                    ┌───────────────┐
                    │   Sensors     │
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │  Perception   │---------------
                    └───────┬───────┘               |  
                            ↓                       |
                ┌──────────────────────────────┐    |
                │ Neural Newtork && Prediction │    |
                └───────────┬──────────────────┘    |
                            ↓                       |
                    ┌───────────────┐               |
                    │ Local Planner │<---------------
                    └───────┬───────┘
                            ↓
                    ┌───────────────┐
                    │ Controller    │
                    └───────────────┘


### Sensors 
#### Lidar -> point cloud mimarisi kullanılacak

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
                3D Bounding Boxes
                ↓
                Local Occupancy Grid
                ↓
                Collision Checking
                ↓
                Local Planner
                ↓
                cmd_vel

-   Bu kavramları kendim öğrenip kendim uygulayacağım, sonrasında AutoWare github sayfasından bu problemler nasıl çözülmüş onları inceleyeceğim.

#### RGB Kamera

-   Görüntü üstünden Engellerin pozisyonları tespit edilip hepsinin konmları çıkarılacak.


### Neural Newtork && Prediction
#### LSTM
-   LSTM kullanılarak elde ettiğimiz veriler birleştirilip multi-modal bir yörünge üretimi yapılacaktır. 

#### Transformers
-   Transformers yapısı kullanılarak elde edilen bilgiler encode ediliğ, attention mekanizması ile birleştirilecekler. 

#### CVAE
-   Conditional Varietional Auto Encoder ile multi model learning yapılacaktır. 

### Local Planner
- DWA ve MCP gibi modeller local planner kısmında kullanılıcaktır. Yörünme üretiminde veril geldiyse bu veriler eşliğinde yerel planlama yapılacktır, aksi taktirde modellerin kendi matematiksel modellemesi ile planlama yapması beklenmektedir.