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

-   [] ROS2, GAZEBO ortamları kurulup, simülasyona hazır hale getirilecekler.
    *   ros2_ws altında ros ortamı kurulacak ve GAZEBOya hem dünya yüklenecek ve sağlıklı şekilde ros ile iletişim              kurabilmesi sağlanacak. Proje ortasında gazebo ile sorun yaşamak istemiyoruz.

-   [] Lidar ve Kamera sensörleri kurulacak; point cloud konusu çalışılacak ve son haline getirilecek.
    *   Lidar ve Kamera sensörleri projeye dahil edilip onlardan veri alınacak. Lidardan alınan veriler Point Cloud olarak işlenecek ve Point cloudlar veri işlemesinden geçirilecek. Bu konu ayrıca çalışılacak o yüzden bu kızım uzun bir süre alacak. Lidardan sonra Kameradan da veri alınıp sensörlerin doğru çalıştığı doğrulanacak ve zamanı gelince bir fonksiyon olarak çağırılıp kolayca kullanılacaklar. 

-   []  Kamera üstünden obje tespit, takip ve ID atama işlemleri yapılacak 
    *   Kameradan gelen veriye nesne tepsit, takip ve ID atama algoritmaları uygulanacak. Buradan elde ettiğimiz veriler saklanacak ve sensör füzyonunda kullanılıp nihai sonuçlara ulaşılacak.

-   [] Lidar ve kamera füzyonu yapılacak
    *   veriler alınıp ön işlemeden geçildikten sonra iki veriden gelen nesne konumları kullanılacak EKF ile bir sensör füzyonu uygulanacak. 

-   [] SLAM ve Base model uygulaması ve testleri
    *   Slam projede temel konumlandırma ve haritalama algoritması olduğundan en başında kurulacak 
    *   İlk model olan Base model ilk defa uygulanıp test edilecek

-   [] LSTM ve CVAE konuları çalışılıp projeye entegre edilecek
    *   İkinci model olan LSTM ve CVAE modeli araştırılıp implemente edilecek ve yeniden benchamark testleri yapılacak

-   []  Transformers ve CVAE projeye entegre edilip testler yapılacak.
    *   Transofmers ve CVAE birleştirilip son model çalıştırılacak ve benchmark testleri yapılacak

-   [] Arayüz geliştirilecek.
    *   Hem konum atama, hem araç poziyon takip etme hem de araç bilgilerini görmek içn interaktif bir arayüz geliştirilecek. Bu arayüz fiziksel araç üstünde kullanılmak üzere geliştirilecektir. 

-   [] Gerçek araç testleri.
    *   Tüm modeller simülasyon üstünde test edildiği gibi gerçek hayatta da test edilecekler ve simülasyon-gerçek arası fark burada gözlenecek. 