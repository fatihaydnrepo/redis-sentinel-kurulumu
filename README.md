# Redis Best Practises

## Redis Sentinel 

Redis Sentinel, Redis Cluster'ı yönetmek ve yüksek kullanılabilirliği sağlamak için kullanılan bir araçtır. Redis Sentinel, Master node' unun düşmesi durumunda otomatik olarak yedek node'un  Master node'unun yerini almasını sağlar. 
![enter image description here](https://raw.githubusercontent.com/ReptilianusBileciktus/redis/main/kurulum%20photo/redis1.png)

Redis Sentinel'in kullanımı ve püf noktaları aşağıdaki gibidir: 

1. Sentinel Konfigürasyonu: Redis Sentinel konfigürasyonu, Redis Cluster'ın yüksek erişilebilirlik özelliklerini sağlamak için çok önemlidir. Sentinel konfigürasyon dosyasını düzenlerken, Master node  ve Sentinel node'unun  doğru şekilde yapılandırılması gerekmektedir. Ayrıca, Sentinel node'un Master node'unun yanıt vermediğinde ne yapacaklarını belirleyen "down-after-milliseconds" ve "failover-timeout" gibi parametreleri ayarlamak da önemlidir.

2.   Sentinel Nodelarının Sayısı: Sentinel nodelarının sayısı, Redis Sentinel'ın doğru şekilde çalışmasını sağlamak için önemlidir. Sentinel node'ları , Master node'unun başarısız olduğunu doğru şekilde tespit etmek için birbirleriyle iletişim kurarlar. Sentinel node'larının sayısı arttıkça, Redis Sentinel'ın doğruluğu ve güvenilirliği de artar. Genellikle, Sentinel node'larının  sayısı çift sayıda olmalıdır.

3.  Sentinel İzleme: Redis Sentinel'ın doğru şekilde çalışmasını sağlamak için, Sentinel node'larının Master node'una düzenli olarak bağlanıp bağlanmadığını izlemek gerekmektedir. Sentinel node'larının düzenli olarak Master node'una bağlanıp bağlanmadığını izlemek, Sentinel node'larının yeniden yapılandırılması veya yeniden başlatılması gerekip gerekmediğini belirlemek için yardımcı olabilir.

4. Redis Sentinel'ın otomatik olarak başlatılması, Redis Sentinel servisinin sistemin her açılışında otomatik olarak başlamasını sağlar. Bu işlem, yüksek erişilebilirliği sağlamak için gereken yedekleme ve yeniden başlatma işlemlerini gerçekleştirmek için önemlidir. Eğer Redis Sentinel servisi otomatik olarak başlatılmazsa, manuel olarak başlatılması gerekebilir ve bu da olası kesinti sürelerine yol açabilir.

	Sistemin açılışında otomatik olarak Redis Sentinel'ın başlatılması için systemd servisleri kullanılabilir. Sistem açılışında servisin otomatik olarak başlaması için bir servis dosyası oluşturulur ve servis dosyası `systemctl enable` komutu ile etkinleştirilir. Böylece, sistem her açıldığında Redis Sentinel servisi otomatik olarak başlar ve yüksek erişilebilirlik süreci kesintiye uğramadan devam eder.

5. Quorum, Sentinel düğümlerinin bir Master node’unun başarısız olduğunu onaylamak için gereken en az Sentinel node sayısını belirler. Quorum ayarları, Redis Sentinel’ın doğru şekilde çalışmasını sağlamak için önemlidir. Quorum ayarını düzenlerken, Sentinel node’larının Master mode’unun başarısız olduğunu doğru şekilde tespit edeceği kadar Sentinel node’una ihtiyaç duyduğundan emin olmalıyız.

###  Redis Sentinel Kurulumu (High Avalibility)

• Redis-Sentinel stable versiyonu indirmek için 
>curl -o redis-stable.tar.gz https://download.redis.io/redisstable.tar.gz

![enter image description here](https://raw.githubusercontent.com/ReptilianusBileciktus/redis/main/kurulum%20photo/11.png)

 • Tar dosyasını dışarı çıkarıyoruz  
 >tar xvf redis-stable.tar.gz
![enter image description here](https://raw.githubusercontent.com/ReptilianusBileciktus/redis/main/kurulum%20photo/Ekran%20g%C3%B6r%C3%BCnt%C3%BCs%C3%BC%202023-03-15%20200303.png)

• İçeride ki trafik internal olacağından güvenlik amaçlı ufw servisini kapatıyoruz 
> sudo systemctl stop ufw
> sudo systemctl disable ufw 

• redis-stable dosyasına giriyoruz
> cd redis-stable

• Aşağıdaki komut ile make edebileceğimiz dosyaları görüntülüyoruz
>ls -lrt

![enter image description here](https://raw.githubusercontent.com/ReptilianusBileciktus/redis/main/kurulum%20photo/12.png)


• make'i indiriyoruz.
> apt install make 

• redis conf'a giriyoruz ve aşağıdaki işlemleri yapıyoruz
> her node'un kendini dinleyeceği şekilde bind ekliyoruz ancak bunu yaparken internal  ip kullanmaya özen göstermelisiniz 
> protected mode no yapıyoruz
> LogFile 'tmp/redis.log' ile loglarımızın toplanacağı yeri belirliyoruz.

• redis conf'da replicaof kısmına master node'muzun ip ve port bilgilerini giriyoruz.
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/16.png?raw=true)

• sentinel.conf içinde aynı işlemleri yapıyoruz.
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/13.png?raw=true)

• sentinel.conf içinde ayrıca sentinel monitor bilgilerimizi giriyoruz. (Bu kısımda rabbitmq-1 yazan yerde normal şartlarda mymaster yazıyor, mymaster yazan yeri rabbitmq-1 makinası olarak belirlediğimiz için rabbitmq-1 adını tüm mymaster'lar için değiştirmemiz gerekmektedir.)
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/14.png?raw=true)

• Yaptığımız ayarları diğer node lara aktarıyoruz. 
>scp /path/to/source_file user@destination:/path/to/destination_directory

Burada `user`, hedef makinedeki kullanıcı adınızı temsil eder. `destination`, hedef makinenin IP adresi veya alan adını temsil eder. Gerekirse, `scp` komutuna -r parametresini ekleyerek dizinlerin ve alt dizinlerin tamamını kopyalayabilirsiniz.

• Aktarım yaptığımız node a geçiş yapıyoruz. ( Burada bind ekledikten so
>cd redis-stable
>nano redis.conf 
	>		- bind ip node  
	>		- replicaof master-ip  master-port(6379)

>nano sentinel.conf
  bind node-ip

• Redis'i başlatmak için her sunucudan parametreleri girmemiz gerekmekte
>cd /redis-stable/src 
> ./redis-server ../redis.conf &
> ./redis-sentinel ../sentinel.conf &

Bu kod parçasında ki kısaca redis sentinel'i bir alt dizindeki sentinel.conf a göre çalıştır ve & ile de bu sürecin loglarını belirlediğimiz log dosyasına kaydetmesi için kullanılmaktadır.
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/45.png?raw=true)


Bu kısımda aşağıdaki komutlar ile log dosyalarında kontrol sağlayarak redis'in düzgün çalıştığını kontrol etmemiz gerekmektedir.
>tail -f /tmp/redis.log
tail -f /tmp/sentinel.log

![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/46.png?raw=true)
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/47.png?raw=true)

Örnek olarak hata aldık ve hatayı düzelterek aşağıdaki komutları çalıştırdık. (buradaki NOAUTH hatası redis.conf dosyasındaki requirepass alanının açık olmasından dolayıdır.)
> Servislerin çalıştığından emin olmak için
>  ps -ef | grep redis
 Eğer durdurulacak servis var ise
  kill -9 pid

![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/48.png?raw=true)

redis.log ve sentinel.log dosyalarından  hata alınmadığında bir sonraki adıma geçebilirsiniz.
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/49.png?raw=true)

• Doğru çalışıp çalışmadığını kontrol etmek için :
>cd /redis-stable
./redis-cli -h master ip -p master-port (6379)

 master node'umuza bağlanarak bir takım verileri işliyoruz ve görüntülüyoruz.
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/55.png?raw=true)


replica node'umuzdan okuma işlemini gerçekleştiriyoruz
![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/ef.png?raw=true)

Sonrasında 10.0.0.4 ip'li master node'umuzu kill ettiğimizde oluşabilecek senaryoyu oluşturacağız.
>ps -ef | grep redis
>kill -9 pid

![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/adwdq.png?raw=true)

Sonrasında diğer makinalardan log dosyalarını kontrol ederek neler olduğunu görebiliriz.
> tail -f /tmp/redis.log

![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/redislog.png?raw=true)

>tail -f /tmp/sentinel.log

![enter image description here](https://github.com/ReptilianusBileciktus/redis/blob/main/kurulum%20photo/senlog.png?raw=true)

Göründüğü üzere diğer 2 node aralarında karar vererek master olarak 10.0.0.5 adresini master olarak seçti.


## Redisson
 Redis için bir Java kütüphanesidir. Bu kütüphane, Redis server'ına erişmek için kullanılır ve Redis özelliklerini kullanarak geliştiricilere bir dizi özellik sunar. Redisson, dağıtılmış sistemler ve uygulamalar için tasarlanmıştır ve bu sayede yüksek performans ve ölçeklenebilirlik sağlar.

Redisson, Redis client kütüphanesi olarak, Redis server'ına bağlanmak, veri okumak, veri yazmak, anahtar değerlerini yönetmek, kuyruklar ve setleri yönetmek gibi birçok Redis özelliğini içerir. Redisson, ayrıca dağıtılmış sistemler için tasarlanmış bazı özelliklere de sahiptir, bunlar arasında aşağıdakiler yer alır:

-   Distributed Map, Multimap ve Set: Redisson, dağıtılmış Map, Multimap ve Set yapısını destekler. Bu yapılar, verilerin farklı sunucular arasında otomatik olarak dağıtılmasını sağlar ve böylece veri depolama kapasitesi ve ölçeklenebilirliği arttırır.
    
-   Distributed Lock ve Semaphore: Redisson, dağıtılmış Lock ve Semaphore özelliklerini destekler. Bu özellikler, birden fazla sunucuda çalışan uygulamalar arasında koordinasyon sağlamak için kullanılabilir.
    
-   Distributed Queue ve Topic: Redisson, dağıtılmış Queue ve Topic özelliklerini de destekler. Bu özellikler, birden fazla sunucu arasında mesajları yönetmek ve dağıtmak için kullanılabilir.
    
-   Live Object: Redisson, Java nesnelerini Redis verilerine dönüştürmek ve bu nesneleri dağıtılmış sistemlerde kullanmak için Live Object özelliğini sunar. Bu özellik, Java nesnelerinin otomatik olarak Redis verilerine dönüştürülmesini sağlar.

Redisson, Redis'in yüksek performansına ve ölçeklenebilirliğine dayanır ve dağıtılmış sistemlerin ihtiyaçlarına cevap vermek üzere tasarlanmış bir kütüphanedir. Redisson'un sahip olduğu özellikler, dağıtılmış sistemlerde uygulama geliştirenler için çok faydalı olabilir.



## Redis Enterprise

Redis Enterprise, Redis Labs tarafından sunulan ve Redis veritabanını işletmelerin kullanımına yönelik bir çözümdür. Redis Enterprise, işletmelerin Redis tabanlı uygulamalarını ölçeklendirmesine ve performansını arttırmasına olanak tanır. Ayrıca, Redis Enterprise ile verilerinizi yedekleyebilir ve iş sürekliliğini sağlayabilirsiniz.

Redis Enterprise, Redis veritabanının açık kaynak kodlu sürümüne ek olarak, işletmelerin ihtiyaçlarını karşılayacak bir dizi özellik sunar. Bu özellikler arasında aşağıdakiler yer alır:

-   Active-Active Geo-Replication: Redis Enterprise, birden fazla bölge arasında verilerin otomatik olarak replikasyonunu sağlar. Bu, işletmelerin dünya çapındaki müşterilerine hızlı ve güvenilir hizmet sunmalarını sağlar.
    
-   High Availability: Redis Enterprise, verilerin yüksek düzeyde mevcudiyetini sağlar. Bu sayede, verilerinizin kaybedilmesi veya erişilemez hale gelmesi riskini minimize edebilirsiniz.
    
-   Data Persistence: Redis Enterprise, verilerin diskte kalıcı olarak depolanmasını sağlayan bir dizi seçenek sunar. Bu, verilerinizi kaybetme riskini azaltır ve iş sürekliliğini sağlar.
    
-   Performance Optimization: Redis Enterprise, performansı artırmak için bir dizi optimizasyon sunar. Örneğin, Redis Enterprise'in Cluster teknolojisi, verilerinizi otomatik olarak dağıtır ve böylece yüksek işlem hacimleriyle başa çıkmanıza yardımcı olur.
    
-   Monitoring and Management: Redis Enterprise, veritabanınızın performansını izlemek ve yönetmek için bir dizi araç sunar. Bu, hata ayıklama ve ölçeklendirme konusunda işletmelerin daha iyi bilgilendirilmesini sağlar.

Redis Enterprise, birçok büyük işletme tarafından kullanılmaktadır ve birçok endüstri standardı tarafından desteklenmektedir. Ayrıca, Redis Labs'ın sürekli geliştirme ve güncelleme çalışmaları sayesinde, Redis Enterprise hızla gelişmektedir ve işletmelerin ihtiyaçlarına uyacak şekilde özelleştirilebilir.

