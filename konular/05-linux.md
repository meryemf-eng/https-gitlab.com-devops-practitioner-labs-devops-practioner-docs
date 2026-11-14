# Linux for DevOps

DevOps dünyasında Linux her yerdedir. Sunucuların ve sanal makinelerin büyük çoğunluğu Linux çalıştırır. CI/CD runner'ları Linux üzerinde koşar ve pipeline adımlarının çoğu aslında birer shell komutudur. Kubernetes node'ları Linux çekirdeğinin özelliklerini kullanır. Bir container ise namespace ve cgroup ile izole edilmiş bir Linux process'inden ibarettir. Docker'ı, Kubernetes'i ve CI'ı anlamanın en kısa yolu Linux'u anlamaktır. Bu doküman, bir DevOps mühendisinin Linux'ta en çok ihtiyaç duyduğu kavramları anlatır.

## Linux mimarisi ve komutların davranışı

Linux'u üç katman olarak düşünebiliriz. En altta donanım vardır: işlemci, bellek, disk ve ağ kartı. Ortada çekirdek (kernel) bulunur; işlemciyi, belleği, dosya sistemini, ağı ve sürücüleri yönetir. En üstte kullanıcı alanı (user space) vardır; shell, servisler ve uygulamalar burada çalışır. Uygulamalar donanıma doğrudan dokunmaz; bir dosya okumak, bir port açmak ya da bellek almak için çekirdekten istekte bulunurlar. Container'lar tam olarak bu sınırda çalışır: aynı çekirdeği paylaşan ama birbirini görmeyen kullanıcı alanları.

Her Linux komutu aynı sözleşmeye uyar: bir process başlatır, girdi alır, çıktı üretir ve sonucunu bir **exit code** ile bildirir. Sıfır başarı, sıfırdan farklı her değer bir hata demektir. Bu küçük ayrıntı otomasyonun temelidir; pipeline'lar ve script'ler bir adımın başarılı olup olmadığını bu koddan anlar. İki çıktı kanalı vardır: **stdout** normal çıktıyı, **stderr** hata ve tanılama bilgisini taşır.

## Dosya sistemi ve izinler

Linux'ta her şey kök dizinden başlar. `/etc` sistem yapılandırmasını, `/var` loglar ve önbellek gibi değişken veriyi, `/home` kullanıcı alanlarını, `/usr` dağıtımın programlarını ve kütüphanelerini, `/opt` ek yazılımları tutar. Bu düzeni bilmek sorun giderirken zaman kazandırır: bir servisin ayarı `/etc` altında, logu `/var/log` altında aranır; disk doluysa ilk şüpheli genellikle `/var`'dır.

Dosya ve dizin işlemlerinde birkaç ayrıntı önemlidir. `ls -lah` izinleri, sahipliği ve boyutu gösterir. `mkdir -p` eksik üst dizinleri de oluşturur. `cp -a` kopyalarken sahiplik ve zaman bilgisini korur; bir yapılandırma dosyasını yedeklerken düz `cp` kullanmak sahipliği değiştirebilir ve servisin dosyayı okuyamamasına yol açabilir. Bir yapılandırma dosyası düzenlendikten sonra söz dizimi, servis durumu ve ilgili log birlikte kontrol edilmelidir.

Linux'ta her dosyanın üç tür kullanıcı için üç izni vardır. Kullanıcılar dosyanın sahibi (owner), ortak erişim grubu (group) ve diğer herkestir (other). İzinler okuma (r), yazma (w) ve çalıştırma (x) izinleridir.

İzinler sayısal olarak da gösterilir: okuma 4, yazma 2, çalıştırma 1 değerindedir. Sahibinin okuyup yazabildiği, grubun yalnızca okuyabildiği ve diğerlerinin hiçbir şey yapamadığı bir dosyanın izni 640'tır; parola içerebilecek bir yapılandırma dosyası için doğru izin budur. İzinler `chmod` ile, sahiplik `chown` ve `chgrp` ile değiştirilir. "Permission denied" hatasında sorun çoğu zaman dosyanın kendisinde değil, yol üzerindeki bir dizinde çalıştırma izninin eksik olmasındadır; bir dizine girebilmek için o dizinde x izni gerekir. `namei -l` komutu bir yolun her dizinindeki izinleri tek tek göstererek bu tür sorunları hızla ortaya çıkarır. "Permission denied" alınca 777 izni vermek ise bir çözüm değil, bir güvenlik açığıdır; doğru yaklaşım kimin okuyacağını, yazacağını ve çalıştıracağını sorup sahipliği ve izinleri buna göre ayarlamaktır.

Yönetici yetkisi `sudo` ile, yalnızca gereken komut ve hedef için verilmelidir. Bir servisi yeniden başlatması gereken birine genel bir yönetici yetkisi değil, sınırlı bir kural verilir; log okuması gereken birine okuma izni verilir. Her sudo kullanımı denetim kayıtlarına yazılır.

Debian tabanlı dağıtımlarda paketler iki araçla yönetilir: depolardan kurulum ve güncelleme için `apt`, yerel paket durumunu sorgulamak için `dpkg`. Script'lerde ve Dockerfile'larda onay sormaması için `apt-get install -y` kullanılır ve paket listesinin önce güncellenmesi unutulmamalıdır.

## Metin işleme ve log okuma

Linux'ta loglar, yapılandırmalar ve komut çıktıları metin olduğu için aynı küçük araçlarla işlenebilir. `grep` satır filtreler, `cut` sütun keser, `sort` ve `uniq` sıralar ve tekrarları sayar, `wc` satır sayar, `sed` bul ve değiştir yapar, `awk` alan bazında işler ve hesap yapar, `tee` çıktıyı hem ekrana hem dosyaya yazar. Bu araçlar **pipe** ile birbirine bağlanır: bir komutun çıktısı sonraki komutun girdisi olur. **Redirect** ise çıktıyı bir dosyaya yönlendirir; örneğin yalnızca hataları bir dosyaya yazmak mümkündür.

Bu araçların gücünü bir log analizi gösterir. Bir web sunucusunun erişim logundan en çok istek atan IP adreslerini bulmak için her satırın ilk alanı alınır, sıralanır, tekrarlar sayılır ve büyükten küçüğe dizilir. Burada sıralama adımı şarttır, çünkü `uniq` yalnızca art arda gelen aynı satırları birleştirir. Aynı yöntemle HTTP durum kodlarının dağılımı da birkaç saniyede çıkarılır. Bir olay anında elinizde bir pano olmasa bile, log ve pipe ile ilk kanıtı dakikalar içinde toplayabilirsiniz.

Sorun giderme logla başlar, ama önce hangi logun hangi soruya cevap verdiğini bilmek gerekir. Ubuntu'da genel sistem mesajları `/var/log/syslog`'da, SSH girişleri, sudo kullanımı ve başarısız giriş denemeleri `/var/log/auth.log`'da, çekirdek mesajları ve bellek yetmediğinde process'leri öldüren OOM killer kayıtları `dmesg` çıktısında bulunur. systemd servislerinin logları `journalctl` ile okunur. Container'larda ise loglar standart çıktıya yazılır. Log okurken iki ilke işe yarar: önce zaman penceresini daraltmak ve önce sayıp gruplayıp sonra tek tek okumak.

Bir örnek, logların nasıl birlikte okunduğunu gösterir. Bir web sunucusunun erişim logunda bazı isteklerin 502 döndüğü görülür. Hata logunda ise arkadaki uygulamaya bağlanılamadığı yazar. Bu iki satır, web sunucusunun ayakta olduğunu ama arkasındaki uygulamanın çalışmadığını söyler; bir sonraki adım uygulama servisinin durumuna bakmaktır. 502 hatası neredeyse her zaman "proxy ayakta, arkası değil" anlamına gelir.

## Process'ler ve kaynaklar

Bir uygulama bir ya da daha fazla process olarak çalışır. Her process'in benzersiz bir kimliği (PID) ve onu başlatan process'in kimliği (PPID) vardır. En üstte PID 1 olan systemd bulunur; diğer bütün process'ler onun altında bir ağaç oluşturur.

`ps` anlık bir fotoğraf verir; `top` ve `htop` canlı görünüm sağlar; `lsof` bir process'in açık dosyalarını ve bağlantılarını listeler. Kaynak kullanımı değerlendirilirken dört şey birlikte düşünülür. **CPU**: hangi process tüketiyor? **Bellek**: uygulamanın kendi belleği mi, yoksa Linux'un boş belleği önbellek olarak kullanması mı? Belleğin dolu görünmesi her zaman sorun değildir. **Load**: CPU'yu ya da diski bekleyen iş kuyruğunun uzunluğu; `uptime` çıktısındaki son 1, 5 ve 15 dakikanın load ortalamaları çekirdek sayısıyla karşılaştırılır. **Zaman**: o sırada bir trafik artışı, dağıtım ya da toplu iş var mıydı?

Disk doluluğu yalnızca kapasite sorunu değildir. Her dosya bir **inode** tüketir; milyonlarca küçük dosya varsa diskte yer olsa bile inode biter ve yeni dosya oluşturulamaz. `df -h` kapasiteyi, `df -i` inode kullanımını, `du` ise yeri hangi dizinlerin tükettiğini gösterir. İlginç bir tuzak da "silinmiş ama açık dosya"dır: bir log dosyası silinir ama onu yazan process dosyayı açık tutmaya devam eder; `df` diski dolu gösterirken `du` göstermez. Servisi yeniden başlatmak alanı serbest bırakır.

## systemd ve servisler

systemd, Linux'ta servisleri yöneten sistemdir. Bir servisin nasıl başlayacağı, hangi kullanıcıyla çalışacağı ve çökünce ne olacağı bir **unit** dosyasında tanımlanır. Tipik bir unit dosyası servisin root yerine kendi kullanıcısıyla çalışmasını, ortam değişkenlerini bir dosyadan almasını, çökerse yeniden başlatılmasını ve sistem açılınca otomatik başlamasını belirtir. Bir uygulamayı elle değil systemd ile çalıştırmanın üç faydası vardır: açılışta kendiliğinden başlar, çökünce yeniden kalkar ve logları merkezi olarak journal'a gider.

Servis yönetiminde `systemctl status` durumu ve son log satırlarını, `systemctl cat` unit dosyasının gerçek içeriğini, `systemctl list-units --failed` başarısız servisleri gösterir. `enable --now` hem açılışta başlatmayı ayarlar hem de servisi hemen başlatır. `restart` servisi durdurup başlatırken `reload` yapılandırmayı kesintisiz yeniden okur. Unit dosyası değiştiğinde `systemctl daemon-reload` ile systemd'ye haber vermek gerekir; bunu unutmak en sık yapılan hatadır. `journalctl -u` bir servisin loglarını okur ve zaman, açılış ve öncelik filtreleri ile daraltılabilir.

Zamanlanmış işler için iki yöntem vardır. Klasik **cron** beş alanlı bir zaman ifadesiyle çalışır ve basit kullanıcı işleri için uygundur. **systemd timer** ise aynı adlı bir servis dosyasını tetikler; loglarının journal'da olması, kaçırılan çalışmaları telafi edebilmesi ve bağımlılık tanımlayabilmesi nedeniyle sunucu ve servis işleri için daha uygundur.

## Ağ ve güvenli erişim

Bir makinenin ağa çıkabilmesi için arayüzün, IP adresinin, yönlendirme tablosunun ve ağ geçidinin birlikte doğru olması gerekir. `ip addr` arayüzleri ve adresleri, `ip route` yönlendirme tablosunu gösterir. Bir portun dinlenip dinlenmediğini `ss -tlnp` gösterir; ama dinlenen bir port dışarıdan erişilemeyebilir. Üç şey kontrol edilir: portu hangi process dinliyor, hangi adreste dinliyor ve firewall izin veriyor mu? Bind adresi özellikle önemlidir: 127.0.0.1'de dinleyen bir servis yalnızca makinenin içinden, 0.0.0.0'da dinleyen bir servis tüm arayüzlerden erişilebilir. Bu ayrım Docker ve Kubernetes'te de sık karşılaşılan hataların kaynağıdır.

Bağlantı sorunlarında aşağıdan yukarı bir merdiven izlenir.

Önce arayüzün açık olduğu ve bir adresi olduğu, sonra ağ geçidine ve hedefe bir yol olduğu, sonra ismin doğru adrese çözüldüğü, sonra servisin dinlediği ve firewall'un izin verdiği, en son da uygulamanın doğru cevap döndüğü kontrol edilir. Bir basamak başarısızsa üsttekilere bakmaya gerek yoktur. Bir uyarı da eklemek gerekir: ping'in çalışmaması her zaman ağın olmadığı anlamına gelmez, çünkü birçok ağda ICMP engellenir. Bu yüzden port seviyesinde doğrulama yapılmalıdır.

**SSH** anahtar tabanlı çalışır. İstemcide gizli kalan bir özel anahtar, sunucuda ise ilgili kullanıcının `authorized_keys` dosyasında bir açık anahtar bulunur. Bağlanırken istemci özel anahtarla bir imza üretir, sunucu açık anahtarla bu imzayı doğrular; özel anahtar hiçbir zaman ağdan geçmez. Bir de **host key** vardır: bu kez sunucu kendini istemciye kanıtlar ve parmak izi değişirse SSH uyarı verir.

SSH'ı doğru kurmak için modern bir ed25519 anahtarı üretilir, açık anahtar sunucuya gönderilir ve sık kullanılan sunucular için kısa adlar tanımlanır. Sunucu tarafında parola ile giriş ve root ile doğrudan giriş kapatılır. Bu tür bir değişiklik yapılırken açık oturum kapatılmamalı, yeni bir pencereden giriş test edilmelidir; aksi hâlde sunucunun dışında kalınabilir. Anahtar dizininin 700, özel anahtarın 600 izinli olması gerekir; daha geniş izinlerde SSH anahtarı reddeder. Dosya kopyalamak için tek dosyada `scp`, dizin eşitlemede ve tekrarlanan kopyalarda yalnızca değişen kısımları gönderen `rsync` tercih edilir.

## Bash ile otomasyon

Elle yapılan işi tekrarlanabilir hâle getirmenin ilk adımı bir shell script'idir. İyi bir script üç satırla başlar. İlk satır, script'i hangi yorumlayıcının çalıştıracağını söyleyen shebang'dir. İkincisi `set -euo pipefail` satırıdır: `-e` bir komut başarısız olursa script'i durdurur, `-u` tanımsız bir değişken kullanıldığında hata verir, `pipefail` ise bir pipe zincirindeki herhangi bir komutun hatasını zincirin hatası sayar. Bu satır her hatayı yakalamaz, ama en sık hataları önleyen bir emniyet kemeridir. Tanımsız değişken korumasının neden önemli olduğunu klasik bir örnek gösterir: `rm -rf "$DIR/"` satırında DIR boş kalırsa komut kök dizini hedef alır.

İyi bir otomasyon script'i eşiklerini ortam değişkenlerinden alır, her kontrolü kısa ve okunur tutar ve sonucunu exit code ile bildirir; böylece bir zamanlayıcı, bir izleme aracı ya da bir pipeline bu koda bakarak karar verebilir. `shellcheck` gibi araçlar script'lerdeki hataları çalıştırmadan bulur.

## Sorun giderme yaklaşımı

Linux'ta sorun gidermek çok sayıda komut çalıştırmak değil, belirtiyi daraltıp her adımda doğrulanabilir kanıt toplamaktır. Önce belirti netleştirilir: kullanıcı ne yaşıyor, ne zamandan beri? Sonra kapsam belirlenir: tek bir makine mi, tek bir servis mi? Sonra kanıt toplanır: log, disk, process, port, DNS. En son yapılan değişikliğin etkisi doğrulanır.

Bir sunucuya girildiğinde ilk dakikada bakılacak yerler de bellidir: load ortalaması, çekirdek mesajları, bellek ve takas kullanımı, çekirdek başına ve process başına CPU, disk kullanımı ve ağ trafiği. Önce kaynaklar, sonra servisler kontrol edilir. Sık görülen arızaların ilk bakılacak yeri de çoğu zaman aynıdır: "disk dolu" hatasında kapasite ve inode, başlamayan bir serviste servis durumu ve logu, "permission denied" hatasında yol üzerindeki izinler, erişilemeyen bir portta bind adresi ve firewall, çözülemeyen bir isimde DNS ayarları, aniden ölen bir process'te ise OOM killer kayıtları.

> Linux, DevOps araçlarının hepsinin zeminidir. Exit code otomasyonun dili, izinler güvenliğin ilk katmanı, systemd servislerin yaşam döngüsü, teşhis merdiveni de ağ sorunlarının yol haritasıdır. Container'ı anlamak, process'i anlamakla başlar.
