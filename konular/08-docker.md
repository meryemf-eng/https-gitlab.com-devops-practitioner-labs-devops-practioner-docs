# Docker ve container

"Benim makinemde çalışıyordu" cümlesi, yazılım dünyasının en eski şikâyetlerinden biridir. Geliştiricinin bilgisayarında, test sunucusunda ve üretimde farklı işletim sistemi sürümleri, farklı runtime'lar, farklı kütüphaneler ve farklı ayarlar vardır. Container'lar bu soruna cevap olarak yaygınlaştı: uygulamayı bağımlılıklarıyla birlikte, her yerde aynı çalışan bir pakete dönüştürdüler. Bu dokümanda container'ın ne olduğunu, Docker'ın nasıl çalıştığını, image'ların nasıl üretildiğini, ağ ve depolamayı, Compose'u ve üretime hazır container pratiklerini ele alıyoruz.

## Container ve sanal makine

Uygulama dağıtımı dört aşamada evrildi. Fiziksel sunucularda uygulamalar aynı işletim sisteminde kütüphane ve kaynak paylaşıyordu; birinin güncellemesi diğerini bozabiliyordu. Sanal makinelerle her iş yükü kendi işletim sistemiyle izole edildi, ama her sanal makine tam bir işletim sistemi taşıdığı için gigabaytlarca disk, yüzlerce megabayt bellek ve dakikalar süren açılış gerektiriyordu. Container'larla uygulama ve bağımlılıkları paketlendi ve çekirdek kontrollü biçimde paylaşıldı. Son aşamada ise CI'ın ürettiği aynı değişmez image testten üretime taşınır oldu.

İkisi arasındaki kritik fark şudur: container host'un çekirdeğini paylaşır, sanal makine ise kendi işletim sistemini ve çekirdeğini taşır. Bu yüzden container'lar megabaytlarla ölçülür ve saniyeler içinde başlar; sanal makinelerin izolasyonu daha güçlüdür ve farklı bir çekirdek çalıştırabilirler. Docker sanallaştırmanın yerini almadı; bulutta container'lar çoğu zaman yine bir sanal makinenin içinde çalışır ve iki teknoloji birbirini tamamlar.

Container'lar dört somut soruna cevap verdi. Ortam uyuşmazlığı: kod, runtime, kütüphane ve ayar aynı paketin içinde taşınır. Yavaş hazırlık: container saniyeler içinde başlar. Kaynak israfı: aynı makinede onlarca hafif iş yükü çalışabilir. Bağımlılık çakışması: her container kendi bağımlılıklarını izole taşır.

## Image ve container

İki temel kavram vardır. **Image**, uygulamanın çalıştırılabilir paketidir: kod, bağımlılıklar ve kullanıcı alanı bileşenleri. Çekirdek image'ın parçası değildir. Image değişmezdir, salt okunurdur, katmanlardan oluşur, bir registry'de saklanır ve bir etiket (tag) ya da içerik özeti (digest) ile tanımlanır. **Container** ise bu image'dan oluşturulmuş çalışan bir örnektir: süreçleri, üzerine yazılabilir bir katmanı, ağı ve bağlı depolama alanları vardır. Bir image'dan istenen kadar container başlatılabilir. Benzetmeyle söylersek image bir sınıf, container o sınıftan oluşturulmuş bir nesnedir; ya da image bir tarif, container pişmiş yemektir. Container silindiğinde yazılabilir katmanı da silinir; bu yüzden container'ın içine yazılan veri, container ile birlikte kaybolur.

Bu yaklaşımın altın kuralı "build once"tır: image bir kez üretilir ve aynı değişmez artifact ortamlar arasında taşınır.

Docker container fikrini icat etmedi; onu herkesin kullanabileceği hâle getirdi. 1979'daki chroot bir process'e ayrı bir kök dizin veriyordu. 2008'de LXC namespace ve cgroup ile Linux container'larını getirdi. 2013'te Docker, image kavramı ve basit bir komut satırı aracıyla bunu yaygınlaştırdı. 2015'te kurulan Open Container Initiative (OCI) image, runtime ve dağıtım için açık standartlar tanımladı. Bu standartlar sayesinde Docker ile üretilen bir image Podman'da ve Kubernetes'te çalışır, farklı runtime'lar birbirinin yerine geçebilir ve Docker Hub, Harbor ve ECR gibi registry'ler aynı protokolü konuşur. Kubernetes'in 2022'de Docker Engine desteğini (dockershim) kaldırması bu yüzden image'ları etkilemedi; Docker ile üretilen image'lar Kubernetes'te çalışmaya devam ediyor.

## Docker'ın mimarisi

Docker istemci-sunucu modeliyle çalışır. Komut satırı aracı (CLI) yalnızca bir istemcidir; komutlar Docker daemon'una, yani dockerd'ye bir API isteği olarak gider. Daemon image'ları, container'ları, ağları ve volume'ları yönetir; registry ise image'ların saklandığı ve dağıtıldığı yerdir.

Bu modelin bir güvenlik sonucu vardır: daemon'un soketine erişebilen herkes daemon'u yönetebilir ve dolayısıyla host üzerinde root yetkisi kazanabilir. docker grubuna yalnızca güvenilir kullanıcılar eklenmelidir. Bir başka önemli ayrıntı, Docker'ın yayınlanan portlar için kendi firewall kurallarını yazmasıdır; bu kurallar ufw gibi host firewall kurallarını atlayabilir. Bir portu yalnızca makinenin içinden erişilebilir yapmak için 127.0.0.1 adresine bağlamak gerekir.

`docker run` dendiğinde arka planda bir zincir çalışır: CLI isteği dockerd'ye gönderir, dockerd image'ın yerelde olup olmadığına bakar ve yoksa registry'den çeker, image'ın üzerine yazılabilir bir katman oluşturur, ağı ve volume'ları bağlar ve process'i başlatır. Runtime tarafında dockerd üst seviye yönetimi yapar, containerd container'ın yaşam döngüsünü yönetir, runc OCI runtime'ı olarak container process'ini oluşturur ve çıkar, containerd-shim ise process ile containerd arasında köprü olarak kalır. Bu sayede dockerd yeniden başlasa bile container'lar çalışmaya devam eder.

## Container izolasyonu

Container'ın izolasyonu bir hypervisor'dan değil, Linux çekirdeğinin dört özelliğinden gelir. **Namespace'ler** process'in görünümünü izole eder: kendi process numaraları, kendi ağı, kendi dosya sistemi görünümü ve kendi makine adı. **Cgroup'lar** CPU, bellek ve process sayısını sınırlar ve ölçer. **Capabilities**, root yetkilerini küçük parçalara böler ve container'a yalnızca gerekenlerin verilmesini sağlar. **Seccomp ve güvenlik modülleri** ise hangi sistem çağrılarının yapılabileceğini kısıtlar.

Bunun pratik bir sonucu vardır: container bir Linux process'idir. Bir nginx container'ı başlatıldığında, container'ın içinden bakıldığında PID 1 olan process, host'tan bakıldığında sıradan bir numarayla görünür; aynı process, iki numara. Container'ı izlemek ve sınırlamak, process'i izlemek ve sınırlamakla aynı şeydir.

Container kendi çekirdeğini getirmediği için uyumlu bir çekirdek gerektirir. Linux'ta Linux container'ları doğrudan çekirdeği kullanır. macOS'ta Docker Desktop arka planda küçük bir Linux sanal makinesi çalıştırır ve container'lar onun içinde koşar. Windows'ta da Linux container'ları için WSL2 ya da bir sanal makine gerekir; Windows container'ları ise Windows çekirdeğinde doğrudan çalışır.

## Container yaşam döngüsü ve operasyon

Bir container oluşturulur (created), başlatılır (running), dondurulabilir (paused), durdurulur (exited) ve silinir (removed). Container bir process olduğu için ana process biterse container da durur. İşini bitirip çıkan bir container'ın exited durumuna geçmesi normaldir; "container'ım hemen kapanıyor" şikâyetinin sebebi çoğu zaman ana process'in bir servis değil, tek seferlik bir komut olmasıdır. Yeniden başlatma politikaları bu davranışı değiştirir: hiç yeniden başlatmamak, yalnızca hata olursa, her durumda ya da elle durdurulmadıkça yeniden başlatmak.

Bir container durduğunda bıraktığı **exit code** ne olduğunu çoğu zaman söyler. 0 normal bitiş, 1 uygulama hatası, 125 Docker'ın container'ı başlatamaması, 126 komutun çalıştırılamaması, 127 komutun bulunamamasıdır. 128'in üzerindeki kodlar sinyal numarasıyla hesaplanır: 137, 128 artı 9 yani SIGKILL ile öldürülmeyi; 143, 128 artı 15 yani SIGTERM ile durdurulmayı gösterir. 137 çoğu zaman container'ın bellek sınırını aşıp çekirdeğin OOM killer'ı tarafından öldürüldüğünü gösterir ve bu container'ın durum bilgisinden doğrulanabilir. Aynı kodlar Kubernetes'te de karşımıza çıkar.

Sınırı verilmeyen bir container host'un bütün belleğini ve işlemcisini kullanabilir. Bellek sınırı sert bir sınırdır: aşıldığında process beklemeden öldürülür. CPU sınırı ise yumuşaktır: aşıldığında process yavaşlatılır ama ölmez. Process sayısı sınırı da kontrolsüz çoğalmaya karşı korur. Bu sınırları çekirdeğin cgroup'ları uygular; Kubernetes'teki requests ve limits da aynı mekanizmanın orkestratör seviyesindeki karşılığıdır.

## Image üretmek: Dockerfile

**Dockerfile**, bir image'ın nasıl üretileceğini adım adım tarif eden reçetedir. Temel talimatları şunlardır: base image'ı seçen `FROM`, çalışma dizinini belirleyen `WORKDIR`, dosya kopyalayan `COPY`, build sırasında komut çalıştıran `RUN`, yalnızca build sırasında geçerli değişken tanımlayan `ARG`, çalışma zamanında da geçerli ortam değişkeni tanımlayan `ENV`, dinlenen portu bilgi olarak belirten `EXPOSE` ve process'in hangi kullanıcıyla çalışacağını belirten `USER`. Her talimatın build zamanını mı, çalışma zamanını mı etkilediğini ayırt etmek tasarımı kolaylaştırır.

`EXPOSE` sık yanlış anlaşılır: port açmaz, yalnızca "bu uygulama bu portu dinliyor" bilgisini taşır. Dışarıdan erişim, container çalıştırılırken portun yayınlanmasıyla oluşur. Dış erişim gerekmeyen bir servisin, örneğin bir veritabanının, portu yayınlanmamalıdır; aynı ağdaki servisler ona doğrudan erişebilir. `CMD` ve `ENTRYPOINT` de karıştırılır: `CMD` varsayılan komutu ya da argümanları sağlar ve çalıştırırken kolayca değiştirilebilir; `ENTRYPOINT` container'ın ana programını sabitler ve verilen parametreler ona argüman olarak eklenir. Yaygın kullanım ikisini birlikte kullanmaktır.

Bir image katmanlardan oluşur: en altta base image, üstünde her Dockerfile adımının ürettiği salt okunur katmanlar, container çalıştığında ise en üstte yazılabilir bir katman.

Katmanlar paylaşılır: aynı base image'ı kullanan on image, o katmanı diskte bir kez tutar. Katmanlar ayrıca yeniden kullanılır: değişmeyen bir adım önbellekten gelir. Ama bir adım değişirse, ondan sonraki bütün adımlar yeniden çalışır. Bu yüzden Dockerfile'daki sıra önemlidir.

Doğru sıralı bir Node.js Dockerfile'ında önce yalnızca package.json ve lockfile kopyalanır ve bağımlılıklar kurulur, kodun geri kalanı ondan sonra kopyalanır. Böylece kodda yapılan küçük bir değişiklik bağımlılık katmanını bozmaz ve build saniyeler içinde biter. Ters sırada yazılan bir Dockerfile ise her kod değişikliğinde bütün bağımlılıkları yeniden indirir.

`docker build .` komutundaki nokta, Docker'a gönderilen **build context**'tir; bu dizinde ne varsa build'e taşınır. **.dockerignore** dosyası neyin gönderilmeyeceğini söyler: Git geçmişi, bağımlılık klasörleri, ortam dosyaları, loglar. Bu dosya build'i hızlandırır, secret dosyalarının yanlışlıkla image'a girmesini engeller ve alakasız dosya değişikliklerinin önbelleği bozmasını önler.

**Multi-stage build**, küçük ve güvenli image üretmenin temel tekniğidir. Dockerfile'da iki aşama bulunur: ilk aşama derleyicilerin ve geliştirme bağımlılıklarının bulunduğu bir image'da uygulamayı derler ve test eder; ikinci aşama temiz bir image ile başlar ve yalnızca çalışan çıktıyı kopyalar. Derleyiciler ve test araçları üretim image'ına girmez. Go gibi statik ikili dosya üreten dillerde boyut farkı çarpıcıdır; tek aşamalı bir image yüzlerce megabayt iken çok aşamalı olanı birkaç megabayta inebilir.

Base image seçimi boyutu, açık sayısını ve hata ayıklama kolaylığını birlikte belirler. Ubuntu tanıdıktır ama gereksiz paketler ve daha fazla açık taşır. Debian slim dengelidir. Alpine çok küçüktür ama farklı bir C kütüphanesi kullandığı için bazı kütüphanelerde uyumsuzluk çıkabilir. Distroless image'larda shell bile yoktur; saldırı yüzeyi küçüktür ama hata ayıklamak zordur. Scratch ise tamamen boştur ve yalnızca statik ikili dosyalar için uygundur. Küçük image daha hızlı indirilir, daha az açık taşır ve daha hızlı ölçeklenir.

İyi bir Dockerfile'ın kuralları şöyle özetlenebilir: sürümü sabitlenmiş bir base image kullanmak, önce bağımlılık dosyalarını sonra kodu kopyalamak, multi-stage build kullanmak, root olmayan bir kullanıcı tanımlamak, .dockerignore kullanmak ve bir sağlık kontrolü eklemek. Kaçınılması gerekenler de bellidir: secret'ları ENV, ARG ya da COPY ile image'a koymak, çünkü image katmanlarında kalırlar; uzaktan dosya indirmek için ADD kullanmak; her komut için ayrı bir RUN satırı yazmak ve build'i tekrarlanamaz yapan genel paket güncellemeleri. Bu kuralların çoğu hadolint gibi bir linter ile otomatik kontrol edilebilir.

## Registry ve image etiketleri

Bir image'ın tam adresi registry, proje, repository ve etiketten oluşur. Registry yazılmazsa Docker Hub, etiket yazılmazsa `latest` varsayılır. `latest` adı yanıltıcıdır; "en yeni" değil, yalnızca "etiketsiz olan" anlamına gelir. Etiketler değişebilir; aynı etiket başka bir image'ı gösterecek şekilde taşınabilir. İçerik özeti, yani digest ise değişmez. Bu yüzden üretimde sürüm etiketleri ya da digest kullanılır; "üretimde hangi sürüm çalışıyor?" sorusunun cevabı her zaman net olmalıdır.

Registry seçenekleri arasında varsayılan registry ve resmi image'ların evi olan Docker Hub, kod platformlarıyla bütünleşik GitHub ve GitLab registry'leri, bulut sağlayıcılarının IAM ile erişilen registry'leri ve kurumun kendi sunucusuna kurduğu Harbor gibi seçenekler vardır. Docker Hub'a giriş yaparken parola yerine kapsamı sınırlı ve iptal edilebilir bir erişim token'ı kullanılır. Docker Hub anonim ve ücretsiz kullanıcılara saatlik indirme sınırları uygular; CI'da giriş yapmamak ya da bir önbellek kullanmamak pipeline'ların bu sınıra takılmasına yol açar. Harbor ve registry konusunu ayrı bir konuda ele alacağız.

## Ağ

Docker'da ağ sürücüsü, container'ın nasıl izole edileceğini ve dış dünyayla nasıl konuşacağını belirler. **bridge** tek bir makinede varsayılan sürücüdür. **host** sürücüsünde container makinenin ağını doğrudan kullanır; ayrı bir adres ve port eşlemesi yoktur, performans yüksek ama izolasyon düşüktür. **overlay** birden çok makineyi tek bir sanal ağda birleştirir. **none** sürücüsünde container'ın yalnızca loopback arayüzü vardır.

Tek bir makinedeki çoklu container uygulamaları için önerilen model, **kullanıcı tanımlı bir bridge ağıdır**.

Bu ağın en büyük avantajı gömülü DNS'tir: aynı ağdaki container'lar birbirini ismiyle bulur. Bir API container'ı veritabanına "db" adıyla erişir ve container'ın IP adresi her yeniden oluşturmada değişse bile isim sabit kalır. Varsayılan bridge ağında ise bu özellik yoktur. Ağlar aynı zamanda bir sınırdır: farklı ağlardaki container'lar birbirine erişemez, bu da ön yüz ile veritabanını ayrı ağlara bölmeyi sağlar.

Container ağı sorunlarının üç klasik sebebi vardır. "Connection refused" genellikle uygulamanın 0.0.0.0 yerine 127.0.0.1'i dinlemesinden kaynaklanır; container içinde loopback adresi yalnızca container'ın kendisidir. "Could not resolve" container'ların aynı kullanıcı tanımlı ağda olmamasından, "port is already allocated" ise host portunun başka bir process tarafından kullanılmasından kaynaklanır.

## Depolama

Container'ın yazılabilir katmanı geçicidir. Kalıcı ya da paylaşılan veri için üç bağlama türünden biri seçilir. **Volume**, Docker'ın yönettiği kalıcı veri alanıdır; container silinse de volume ve veri kalır ve yeni bir container aynı volume'u bağlayarak eski veriye erişir. Veritabanları ve kalıcı uygulama verisi için doğru seçim budur. **Bind mount**, makinedeki belirli bir dizini doğrudan container'ın içine bağlar; makinede yapılan değişiklik anında container'da görünür. Yerel geliştirmede ve yapılandırma dosyalarını vermek için kullanılır, ama makinenin dosya yapısına bağımlıdır. **tmpfs** ise veriyi diskte değil bellekte tutar; container durunca içerik kaybolur ve geçici ya da hassas veriler için uygundur.

Volume'un yaşam döngüsü container'dan bağımsızdır; bir veritabanı container'ı silinip yeni sürümüyle yeniden oluşturulduğunda aynı volume'a bağlanır. Ama volume'un kalıcı olması yedekli olduğu anlamına gelmez; disk bozulabilir ya da volume silinebilir. Yedek, kısa ömürlü bir yardımcı container ile alınır ve geri yüklemesi mutlaka denenir; geri yüklenmeyen yedek yedek sayılmaz. Canlı bir veritabanının dosyalarını kopyalamak tutarsız bir yedek üretebilir; bunun yerine veritabanının kendi yedekleme araçları kullanılmalıdır. Bind mount'larda sık görülen "permission denied" sorunu ise container içindeki kullanıcı numarasının makinedeki dizinin sahibiyle uyuşmamasından kaynaklanır. Yapılandırma ve sertifika dosyalarını salt okunur bağlamak da iyi bir pratiktir.

## Docker Compose

Container'ları tek tek uzun komutlarla başlatmak, ağları ve volume'ları ayrı ayrı oluşturmak hızla zahmetli hâle gelir. **Docker Compose**, bir uygulamanın bütün bileşenlerini tek bir YAML dosyasında tanımlar: çalışacak servisler, servisler arası ağlar, kalıcı volume'lar, ortam değişkenleri ve başlangıç sırası ile sağlık kontrolleri. Compose her proje için bir ağ oluşturur ve servis adları DNS adı olur.

Compose'da yapılandırmanın üç kanalı vardır ve karıştırılmamalıdır. Değişken yerleştirme ve .env dosyası yerel parametreler için kullanılır ama bir secret deposu gibi düşünülmemelidir. Ortam değişkenleri uygulamanın çalışma ayarlarıdır, ama container bilgisinde ve loglarda görünebilir. Hassas veriler için ise **secrets** kullanılır: yalnızca ihtiyaç duyan servise verilir ve container içinde bir dosya olarak bulunur.

Compose'da sık yaşanan bir sorun, container'ın ayakta olmasının servisin hazır olduğu anlamına gelmemesidir. Basit bir bağımlılık tanımı yalnızca başlangıç sırasını kurar; veritabanı container'ı önce başlar ama henüz bağlantı kabul etmiyor olabilir ve uygulama bağlanamayıp çöker. Çözüm, veritabanına bir sağlık kontrolü eklemek ve uygulamanın bağımlılığını "sağlıklı olduğunda başla" koşuluna bağlamaktır. Bu yarış durumunu yeniden başlatma politikasıyla örtmek doğru değildir; Kubernetes'te aynı fikir readiness probe olarak karşımıza çıkar.

Gerçekçi bir Compose dosyasında web, API ve veritabanı servisleri ile iki ağ bulunur: ön yüz ağında web ve API, arka yüz ağında API ve veritabanı. Böylece web servisi veritabanını göremez ve bir web açığı doğrudan veritabanına ulaşamaz. Compose bir stack'in bütün yaşam döngüsünü yönetir: dosyayı doğrulamak, servisleri başlatmak, durumlarını ve loglarını izlemek, servis içinde komut çalıştırmak ve stack'i kaldırmak. Kaldırırken volume'ları da silen seçeneğin veritabanı verisini de geri dönüşsüz sildiği unutulmamalıdır. Compose yerel geliştirme, entegrasyon testi ve küçük tek makineli dağıtımlar için çok kullanışlıdır; cluster orkestrasyonu ise Kubernetes'in alanıdır.

## Üretime hazır container'lar

Üretime hazır bir container için altı pratik öne çıkar: resmi ya da güvenilir bir base image, küçük bir image, root olmayan bir kullanıcı, image'a gömülmemiş secret'lar, .dockerignore ve zafiyet taraması ile kritik image'larda digest sabitleme. Çalışma anında ise yetkiler bilinçli olarak daraltılır: root olmayan kullanıcı, mümkünse salt okunur kök dosya sistemi, gereksiz Linux capability'lerinin kaldırılması, sonradan yetki kazanmanın engellenmesi, varsayılan seccomp ve güvenlik profillerinin kapatılmaması ve kaynak sınırları. Mantık basittir: bir container ele geçirilirse, saldırganın elinde ne kadar az yetki kalırsa hasar o kadar küçük olur. İmaj taraması ve CI kapısı DevSecOps konusunde ayrıntılı ele alınır.

Container'ları gözlemlemenin kanalları da bellidir: uygulama loglarını dosyaya değil standart çıktıya yazar ve loglar merkezi bir platforma toplanır; kaynak kullanımı, yaşam döngüsü olayları ve sağlık kontrolleri ayrı ayrı izlenir. Sorun giderirken rastgele komut denemek yerine sabit bir sıra izlenir: önce durum, sonra loglar, sonra yapılandırma, sonra ağ, sonra depolama, en son kaynaklar. Son adım her zaman sorunun tekrarını önleyen kalıcı bir değişikliktir; container'ın içine girip elle yapılan bir düzeltme, container yeniden oluşturulduğunda kaybolur.

## Docker'dan Kubernetes'e

Docker ve Compose'da öğrenilen her kavramın Kubernetes'te bir karşılığı vardır. Container'ın karşılığı Pod, Compose servisinin karşılığı Deployment ile Service, servis adıyla DNS'in karşılığı Service DNS, named volume'un karşılığı PersistentVolumeClaim, sağlık kontrolünün karşılığı probe'lar, ortam değişkenleri ve secret'ların karşılığı ConfigMap ve Secret'tır. Dockerfile, OCI image, registry ve değişmez artifact ise aynen geçerli kalır. Kubernetes'in eklediği şey zamanlama, kendi kendini iyileştirme, kademeli güncelleme, yatay ölçekleme ve cluster ağıdır.

> Container, namespace ve cgroup ile izole edilmiş bir Linux process'idir ve host çekirdeğini paylaşır. Image değişmez bir şablondur; doğru sıralı ve çok aşamalı bir Dockerfile küçük ve güvenli image üretir. Kullanıcı tanımlı ağlar isimle erişim, volume'lar kalıcı veri, Compose ise çoklu container uygulamalarını tek dosyada tanımlamayı sağlar.
