# Kubernetes

Docker bir uygulamayı her yerde aynı çalışan bir pakete dönüştürür, ama tek bir makinede. Onlarca makineye yayılmış yüzlerce container'ı hangi makinede çalıştıracağınıza, biri çöktüğünde ne olacağına, yeni sürümü kesintisiz nasıl yayınlayacağınıza ve trafiği nasıl dağıtacağınıza kim karar verecek? Kubernetes bu sorulara cevap veren bir orkestrasyon platformudur. Bu dokümanda Kubernetes'in mimarisini, temel nesnelerini, ağ ve depolama modelini, uygulama sağlığını, ölçeklemeyi, dağıtım stratejilerini, güvenliği ve Helm'i ele alıyoruz.

## Kubernetes nedir?

Kubernetes, container tabanlı iş yüklerinin dağıtımını, ölçeklenmesini ve yaşam döngüsünü otomatikleştiren bir platformdur. Üç temel özelliği vardır. **Declarative** çalışır: ne istendiği YAML ile tanımlanır, nasıl yapılacağını Kubernetes bulur ve bu tanımlar Git'te sürümlenebilir. **Self-healing**'dir: çöken bir container'ı yeniden başlatır, kaybolan bir makinedeki iş yüklerini başka makinelere taşır ve istenen kopya sayısını sürekli korur. **Platform hizmetleri** sunar: servis keşfi, depolama, politikalar ve otomatik ölçekleme.

Kubernetes bir "container çalıştırma aracı" değil, dağıtık sistemler için bir kontrol ve orkestrasyon katmanıdır. Container'ları yine containerd gibi bir runtime çalıştırır; Kubernetes neyin, nerede, kaç kopya çalışacağına karar verir. Bütün davranışının merkezinde **uzlaştırma** (reconciliation) döngüsü vardır: istenen durum ile gerçek durumu sürekli karşılaştırır ve farkı kapatır.

Kubernetes, Google'ın iç cluster yöneticisi Borg'dan doğdu, 2014'te açık kaynak olarak yayınlandı ve CNCF'in ilk mezun projesi oldu. Yılda üç sürüm çıkar ve her sürüm yaklaşık on dört ay desteklenir; bu, cluster'ların düzenli olarak güncellenmesi gerektiği anlamına gelir. Kurulum seçenekleri öğrenme ve CI için kind ya da minikube gibi yerel araçlardan, kubeadm ile kurulan ve işletimi kuruma ait cluster'lara ve kontrol düzlemini bulut sağlayıcının yönettiği EKS, AKS ve GKE gibi yönetilen servislere kadar uzanır.

## Mimari

Bir Kubernetes cluster'ı iki katmandan oluşur: karar veren **control plane** ve iş yükünü çalıştıran **worker node**'lar.

Control plane'in bileşenleri şunlardır. **kube-apiserver** API'nin giriş noktasıdır; bütün bileşenler birbiriyle onun üzerinden konuşur. **etcd**, cluster'ın bütün durumunu ve ayarlarını tutan anahtar-değer veritabanıdır; etcd'nin kaybı cluster'daki bütün tanımların kaybı demektir ve bu yüzden yedeklenmesi hayatidir. **kube-scheduler** henüz bir node'a atanmamış Pod'lar için uygun bir node seçer. **kube-controller-manager** controller döngüleriyle istenen durumu korur. **cloud-controller-manager** ise bulut sağlayıcıyla entegrasyonu yönetir; örneğin LoadBalancer tipinde bir Service için bulutta yük dengeleyici açar.

Worker node'larda üç bileşen bulunur. **kubelet**, API server'dan kendi node'una atanan Pod tanımlarını alır ve uygular. **Container runtime** image'ı çeker ve container'ları çalıştırır. **kube-proxy** ise Service trafiği için node seviyesinde ağ kurallarını yazar. Bunlara ek olarak **CNI** eklentisi Pod ağını kurar.

Bir isteğin akışı mimariyi iyi anlatır.

kubectl isteği API server'a gönderir. API server isteği doğrular ve durumu etcd'ye yazar. Controller'lar API'yi izler ve gereken nesneleri oluşturur. Scheduler Pod için bir node seçer. O node'daki kubelet Pod tanımını alır, runtime image'ı çeker ve container'ı başlatır. Bu akışta kimse kimseye doğrudan emir vermez; her bileşen API server'ı izler ve kendi işini yapar. Bu "izle ve uzlaştır" modeli Kubernetes'in dayanıklılığının kaynağıdır. Sık karıştırılan bir nokta da şudur: scheduler container başlatmaz, yalnızca node seçer; container'ları seçilen node'daki kubelet başlatır.

Çekirdek bileşenlerin yanında eklentiler vardır: servis isimlerini çözen **CoreDNS**, kaynak kullanımını ölçüp otomatik ölçeklemeyi besleyen **metrics-server** ve ağ, depolama ve runtime için standart arayüzler olan CNI, CSI ve CRI. Bu arayüzler sayesinde Kubernetes'e farklı ağ, depolama ve runtime eklentileri takılabilir. Üretimde control plane tek bir makinede çalışmaz: API server birden çok control plane node'unda çalışır, önlerindeki bir yük dengeleyici trafiği dağıtır ve etcd üç ya da beş üyeli bir küme olarak çalışır. Yönetilen servislerde bu iş bulut sağlayıcıya aittir.

## kubectl, kubeconfig ve namespace

kubectl hangi cluster'a hangi kimlikle bağlanacağını **kubeconfig** dosyasından öğrenir. Bu dosyada cluster adresleri, kullanıcı kimlik bilgileri ve **context**'ler bulunur; bir context bir cluster, bir kimlik ve varsayılan bir namespace'in birleşimidir. Birden çok cluster ile çalışırken en tehlikeli hata, yanlış context'te komut çalıştırmaktır; bu yüzden hangi context'te olunduğu her zaman kontrol edilmelidir.

kubectl'in komutları aynı kalıbı izler: bir fiil, bir kaynak türü, bir ad ve seçenekler. `get` listeler, `describe` ayrıntıları ve olayları gösterir, `apply` bir manifest'i uygular, `logs` ve `exec` log okur ve container'a girer, `explain` bir alanın dokümantasyonunu verir, `diff` uygulamadan önce farkı gösterir. YAML ezberlemeye gerek yoktur; `kubectl explain` alanları açıklar, `--dry-run=client -o yaml` ise bir komuttan hazır bir manifest iskeleti üretir.

**Namespace**, cluster içinde mantıksal bir bölmedir ve takım ya da ortam sınırı olarak kullanılır. Kubernetes'in kendi bileşenleri kube-system namespace'inde çalışır. Node'lar ve kalıcı volume'lar gibi cluster seviyesindeki kaynaklar ise bir namespace'e ait değildir. Namespace tek başına ağ izolasyonu sağlamaz; farklı namespace'lerdeki Pod'lar varsayılan olarak birbirine erişebilir.

Kubernetes ile iki şekilde çalışılabilir. **Imperative** yaklaşımda "şunu yap" denir; hızlı test ve sorun giderme için uygundur ama değişiklik geçicidir ve geçmişi izlenemez. **Declarative** yaklaşımda "durum böyle olsun" denir; YAML manifest'leri Git'te sürümlenir, incelenir ve tekrarlanabilir. Üretimde değişiklik her zaman declarative yoldan yapılmalıdır.

Her manifest'in dört ana alanı vardır: kaynağın türünü seçen **apiVersion** ve **kind**, isim, etiket ve açıklamaları taşıyan **metadata** ve istenen durumu tanımlayan **spec**. Manifest'te yazılmayan ama Kubernetes'in doldurduğu bir beşinci alan da vardır: gözlenen durumu gösteren **status**. Kubernetes'in bütün işi spec ile status'u aynı yapmaktır.

## Pod, ReplicaSet ve Deployment

**Pod**, Kubernetes'in en küçük dağıtım birimidir. Bir ya da daha fazla container'ı birlikte çalıştırır; bu container'lar aynı node'a yerleşir, aynı ağ alanını ve tek bir IP adresini paylaşır ve gerekirse aynı volume'ları kullanır. Neden doğrudan container değil de Pod? Çünkü bazen birbirine çok sıkı bağlı süreçler gerekir: uygulama ve onun loglarını gönderen bir yardımcı, ya da uygulamadan önce çalışıp yapılandırma hazırlayan bir init container. Pratikte Pod'ların çoğu tek container içerir. Pod'lar doğrudan nadiren oluşturulur, çünkü tek başına bir Pod silindiğinde geri gelmez.

Pod'lar ve diğer nesneler **label**'larla etiketlenir ve **selector**'larla seçilir. Service hangi Pod'lara trafik göndereceğini, Deployment hangi Pod'ları yöneteceğini selector ile bulur. Yanlış bir selector, trafiğin ya da yönetimin kaybolması demektir; Kubernetes'te yaşanan sorunların şaşırtıcı bir kısmı bir etiket yazım hatasıdır. **Annotation**'lar ise seçim için değil, açıklama ve araçlar için kullanılan metadata'dır.

**ReplicaSet**, istenen sayıda Pod'un çalışmasını selector ve label'lar üzerinden sürekli koruyan controller'dır. İstenen sayı üçse ve bir Pod çökerse yenisini oluşturur. Görevi Pod çalıştırmak değil, istenen Pod sayısını korumaktır.

**Deployment** ise ReplicaSet'leri yöneten ve uygulamaların neredeyse her zaman üzerinden dağıtıldığı nesnedir. Üç kritik alanı vardır: istenen kopya sayısı, yönetilecek Pod'ları seçen selector ve yeni Pod'ların şablonu. Selector ile şablondaki label'ların eşleşmesi zorunludur.

Deployment doğrudan container başlatmaz. Deployment controller bir ReplicaSet oluşturur, ReplicaSet controller istenen sayıda Pod nesnesi oluşturur, scheduler her Pod için bir node seçer ve kubelet Pod'u çalıştırır. Her yeni image sürümünde Deployment yeni bir ReplicaSet oluşturur; kademeli güncelleme ve geri alma bu sayede çalışır. `scale` ve `set image` gibi komutlar anlık değişiklik yapar, ama kalıcı değişiklik manifest'te ve Git'te yapılmalıdır; aksi hâlde bir sonraki apply elle yapılan değişikliği geri alır.

## Ağ ve trafik

Kubernetes'in ağ modeli basit kurallara dayanır: her Pod cluster içinde adreslenebilir bir IP alır, aynı Pod'daki container'lar ağ alanını paylaşır ve farklı node'lardaki Pod'lar adres çevirme olmadan birbirine ulaşabilir. Bunu bir CNI eklentisi sağlar: politika ve yönlendirmede güçlü Calico, eBPF tabanlı Cilium ya da Pod'lara doğrudan bulut ağından adres veren bulut CNI'ları. Ama Pod IP'leri geçicidir; Pod yeniden oluştuğunda değişir. Bu yüzden uygulamalar birbirine Pod IP'siyle değil, **Service** adıyla erişir.

Service, değişen Pod'ları soyutlayan sabit bir sanal IP ve DNS adıdır. Selector ile uygun Pod'ları bulur ve trafiği yalnızca hazır olanlara yönlendirir; cluster içindeki bir L4 yük dengeleyici gibi çalışır. Üç temel tipi vardır. **ClusterIP** varsayılan tiptir ve yalnızca cluster içinden erişim sağlar; veritabanları ve iç servisler için doğru seçimdir.

**NodePort** her node'un belirli bir portundan Service'e erişim sağlar; test ortamlarında kullanışlıdır. **LoadBalancer** ise bulut entegrasyonu varsa dışarıda gerçek bir yük dengeleyici açar. Service tanımında üç port kavramı karıştırılmamalıdır: istemcinin geldiği **port**, Pod'un dinlediği **targetPort** ve node'un açtığı **nodePort**. Selector Pod etiketleriyle uyuşmazsa Service'in arkası boş kalır ve bağlantılar reddedilir; ilk teşhis adımı Service'in endpoint listesine bakmaktır.

Servis keşfi DNS ile çalışır. Pod'un DNS ayarları cluster DNS'ine, yani CoreDNS'e yönlendirilmiştir ve CoreDNS servis adlarını ClusterIP'ye çevirir. Bir servisin tam adı servis adı, namespace ve cluster alan adından oluşur; aynı namespace içinde yalnızca servis adı yeterlidir. Bir Pod'a ulaşılamadığında dört soru sorulur: DNS mi, Service mi, Pod mu, yoksa bir ağ politikası mı? Geçici bir teşhis Pod'u açıp DNS ve Service'i denemek, port yönlendirme ile Service katmanını atlayıp doğrudan Pod'a bağlanmak bu soruları ayırt etmenin yollarıdır.

HTTP ve HTTPS trafiğini host ve path'e göre yönlendirmek için **Ingress** kullanılır: tek bir giriş noktasıyla birçok servis yayınlanabilir. Ingress API'si desteklenmeye devam ediyor ama artık geliştirilmiyor. Ayrıca en yaygın Ingress controller olan topluluk projesi Ingress NGINX 2025'te emekliye ayrıldı ve 2026'da bakımı tamamen sona erdi; mevcut kurulumlar çalışmaya devam ediyor ve başka controller'lar Ingress'i destekliyor. Yeni projeler için önerilen yön **Gateway API**'dir.

Gateway API rol ayrımına dayanır. **GatewayClass** cluster seviyesinde hangi uygulamanın kullanılacağını tanımlar ve altyapı ekibinin işidir. **Gateway** dinleyicileri, yani portları ve sertifikaları tanımlar ve platform ekibinin işidir. **HTTPRoute** ise yönlendirme kurallarını tanımlar ve uygulama ekibinin işidir. Bu ayrım, platform engineering yaklaşımıyla çok uyumludur. Gateway API ayrıca ağırlıklı yönlendirme gibi özellikleri standart alanlarla sunar.

## Yapılandırma ve depolama

Kubernetes'in çalıştırdığı her şey bir image'dır ve image etiketi çalışan sürümü belirler. Image özel bir registry'deyse Pod'a o registry'nin kimlik bilgisi bir **imagePullSecret** olarak verilir; verilmezse ImagePullBackOff hatası alınır.

**ConfigMap**, hassas olmayan yapılandırma verisini image'dan ayırır; aynı image farklı ortamlarda farklı ayarlarla çalıştırılabilir. **Secret** ise parola ve token gibi hassas veriler için kullanılan nesnedir. Her ikisi de Pod'a ortam değişkeni ya da bağlanmış bir dosya olarak verilebilir. Ortam değişkenleri Pod başlarken okunur; ConfigMap değiştiğinde çalışan Pod'lar eski değeri kullanmaya devam eder ve yeniden başlatılmaları gerekir. Volume olarak bağlanan ConfigMap'ler ise zamanla kendiliğinden güncellenir.

Secret hakkında yaygın bir yanılgı düzeltilmelidir: Secret'taki değerler yalnızca base64 ile kodlanmıştır, şifrelenmemiştir; base64'ü herkes tek bir komutla çözebilir. Gerçek koruma dört yoldan gelir: API server'ın Secret'ları etcd'de şifreli saklaması, Secret okuma yetkisinin RBAC ile yalnızca ilgili kimliklere verilmesi, Vault ya da bulut secret yöneticisi gibi bir kasadan Secret'ların cluster'a eşitlenmesi ve Secret'ların Git'e şifreli hâlde konup yalnızca cluster'ın çözebilmesi. Düz bir Secret manifest'ini Git'e koymak, parolayı herkese açmak demektir.

Depolama tarafında **geçici volume'lar** Pod'un ömrüyle yaşar; emptyDir bunların en basitidir ve aynı Pod'daki container'ların dosya paylaşmasını sağlar. Pod yeniden oluşturulduğunda verinin hâlâ gerekmesi gerekiyorsa kalıcı depolama kullanılır. **PersistentVolume** (PV), cluster'ın sunduğu ve Pod'dan bağımsız yaşayan bir depolama kaynağıdır; arkasında bir bulut diski, bir NFS paylaşımı ya da bir blok depolama olabilir. **PersistentVolumeClaim** (PVC), uygulamanın depolama talebidir: ne kadar kapasite, hangi erişim modu ve hangi **StorageClass**. StorageClass dinamik provizyonlamayı tanımlar ve bir PVC geldiğinde PV'yi otomatik oluşturur.

Erişim modları önemlidir. **ReadWriteOnce** volume'un tek bir node'dan okunup yazılabildiğini söyler; bulut disklerinin çoğu böyledir. Burada dikkat edilmesi gereken, kısıtın tek Pod değil tek node olmasıdır. **ReadOnlyMany** birçok node'dan yalnızca okunabilen, **ReadWriteMany** birçok node'dan okunup yazılabilen volume'lardır; ikincisi NFS gibi paylaşımlı dosya sistemleri gerektirir. PVC silindiğinde ne olacağını **reclaim policy** belirler: Delete arkadaki diski de siler, Retain ise korur ve elle temizlik ister. Dinamik provizyonlamanın varsayılanı çoğu zaman Delete'tir; üretim veritabanları için bu ayar bilinçli olarak seçilmelidir.

## Uygulama sağlığı

Kubernetes uygulamanın sağlığını üç farklı soruyla kontrol eder ve her soru için ayrı bir **probe** vardır.

**Startup probe** "uygulama başladı mı?" diye sorar; başarılı olana kadar diğer probe'lar çalışmaz. Yavaş açılan uygulamaları, örneğin büyük bir önbellek yükleyen servisleri, erken öldürülmekten korur. **Readiness probe** "trafik alabilir mi?" diye sorar; başarısız olursa container yeniden başlatılmaz, yalnızca Service'in arkasından çıkarılır. Bir bağımlılık geçici olarak kullanılamıyorsa Pod'u trafik dışına almak, onu yeniden başlatmaktan daha güvenlidir. **Liveness probe** "container kurtarılamaz durumda mı?" diye sorar; üst üste başarısız olursa kubelet container'ı yeniden başlatır. Kilitlenmiş ama process'i ayakta olan bir uygulama için uygundur.

Probe'lar yanlış kullanıldığında ciddi sorunlara yol açar. En sık yapılan hata, liveness probe'un veritabanı bağlantısını kontrol etmesidir: veritabanı birkaç saniye yavaşladığında bütün Pod'ların liveness kontrolü başarısız olur, hepsi aynı anda yeniden başlar ve küçük bir sorun bütün uygulamanın kesintisine dönüşür. Doğru tasarımda readiness bağımlılıkları kontrol eder, liveness ise yalnızca process'in kendisini. Liveness için sorulacak soru şudur: bu container kendi kendine toparlanabilir mi? Cevap hayırsa liveness uygundur.

Pod'lar kapanırken de bir sıra izlenir: Pod endpoint listesinden çıkarılır, varsa preStop kancası çalışır ve SIGTERM gönderilir, belirlenen süre boyunca uygulama devam eden işlerini bitirir, süre dolarsa SIGKILL gelir. Uygulamanın SIGTERM'i yakalayıp düzgün kapanması, kademeli güncellemelerde istek kaybını önler.

Pod sorunlarında sabit bir kanıt toplama sırası izlenir: önce Pod'un durumu, sonra describe çıktısındaki olaylar ve koşullar, sonra container logları ve gerekirse bir önceki, yani çöken container'ın logları, sonra cluster olayları, en son etkileşimli hata ayıklama. Durum sütunu sorunun katmanını çoğu zaman söyler:

| Durum | Anlamı | Olası sebep |
|---|---|---|
| Pending | Planlanamadı | Yetersiz kaynak, taint, bağlanmamış PVC |
| ImagePullBackOff | Image çekilemedi | Yanlış etiket, eksik imagePullSecret |
| CrashLoopBackOff | Başlıyor ve çöküyor | Uygulama hatası, eksik ortam değişkeni |
| OOMKilled | Bellek sınırı aşıldı | Düşük sınır ya da bellek sızıntısı |
| CreateContainerConfigError | Yapılandırma bulunamadı | Olmayan ConfigMap ya da Secret |
| Running ama hazır değil | Readiness başarısız | Probe yolu ya da bağımlılık |

## Kaynaklar ve ölçekleme

Her container için iki kaynak değeri tanımlanır. **Requests**, scheduler'ın yerleştirme hesabında kullandığı minimum kaynak beklentisidir; bir node'a ancak requests toplamı sığıyorsa Pod konur. **Limits** container'ın kullanabileceği üst sınırdır; CPU sınırı aşılırsa yavaşlama, bellek sınırı aşılırsa OOMKilled olur. Requests verilmezse node'lar aşırı doldurulabilir; çok yüksek verilirse kaynak israf edilir. Namespace seviyesinde **LimitRange** tek tek nesneler için varsayılan ve sınır değerler, **ResourceQuota** ise namespace'in toplam bütçesini tanımlar; paylaşılan cluster'larda bir takımın bütün kaynakları tüketmesini bunlar engeller.

**Horizontal Pod Autoscaler** (HPA), gözlenen bir metriğe göre kopya sayısını değiştirir. CPU kullanımı requests değerine oranlanarak hesaplanır; bu yüzden requests tanımlı değilse HPA çalışmaz. Formül basittir: istenen kopya sayısı, mevcut kopya sayısı çarpı kullanım bölü hedeftir. Dört Pod yüzde 140 kullanımda ve hedef yüzde 70 ise HPA sekiz Pod'a çıkar. Ölçeği küçültürken ise ani dalgalanmaları önlemek için bir bekleme penceresi uygulanır. HPA'nın çalışması için metrics-server'ın kurulu olması gerekir.

Üç ölçekleme modeli üç katmanda çalışır: HPA uygulama katmanında kopya sayısını, VPA (Vertical Pod Autoscaler) Pod katmanında kaynak değerlerini, Cluster Autoscaler ve Karpenter gibi araçlar ise altyapı katmanında node sayısını ayarlar. Bulutta üçü birlikte çalışır: HPA Pod ekler, Pod'lar sığmayınca node eklenir.

## İleri iş yükleri

Deployment sürekli çalışan durumsuz uygulamalar içindir, ama tek iş yükü türü değildir. **Job**, başarıyla tamamlanması beklenen işler içindir: veritabanı göçleri, veri aktarımları, toplu işler. Bir göçü Deployment ile çalıştırmak, iş bitip container çıktığında Kubernetes'in onu yeniden başlatması ve göçün tekrar tekrar çalışması demektir. **CronJob**, zamanlaması gelince bir Job oluşturur; saat dilimini ve önceki iş bitmeden yenisinin başlayıp başlamayacağını açıkça belirtmek gerekir.

**StatefulSet**, kimlik ve depolama sürekliliği gereken iş yükleri içindir. Pod'lar sıralı ve sabit isimler alır ve her biri kendi PVC'sine bağlanır; bir Pod silinip yeniden oluşturulduğunda aynı isim ve aynı volume ile geri gelir. Bir headless Service ile birlikte kullanılır ve her Pod'a ayrı bir DNS kaydı verir. Ama StatefulSet kullanmak bir veritabanını otomatik olarak kümeli hâle getirmez; çoğaltma ve yedekleme uygulamanın işidir ve üretimde bir operator ya da yönetilen bir veritabanı düşünülmelidir. **DaemonSet** ise uygun her node'da bir Pod çalıştırır ve log toplayıcılar, izleme ajanları ve ağ bileşenleri için kullanılır.

İş yükü seçimi üç soruyla yapılır: iş sürekli mi çalışacak, değilse Job ya da CronJob. Her node'da bir Pod mu gerekli, öyleyse DaemonSet. Sabit kimlik ya da Pod'a özel kalıcı depolama mı gerekli, öyleyse StatefulSet; değilse Deployment.

## Dağıtım stratejileri

Her yeni sürüm bir risktir; dağıtım stratejisi bu riskin ne kadarının, kime ve ne kadar süre yansıyacağını belirler. Strateji seçerken kesintinin kabul edilebilir olup olmadığı, iki sürümün aynı anda çalışıp çalışamayacağı, geri dönüşün ne kadar hızlı olması gerektiği, riskin önce kime gösterileceği, ek kaynak bütçesi ve başarının nasıl ölçüleceği sorulur.

**Recreate** stratejisinde eski sürüm tamamen durur, sonra yeni sürüm başlar; basittir ama kesinti yaratır ve kesintinin kabul edilebildiği ya da iki sürümün birlikte çalışamadığı durumlarda kullanılır. **Rolling update** Kubernetes'in varsayılanıdır: Pod'lar kademeli olarak yeni sürüme geçer.

Deployment yeni ReplicaSet'i büyütürken eskisini küçültür. Geçişin hızını ve güvenliğini iki ayar belirler: geçiş sırasında fazladan açılabilecek Pod sayısı (maxSurge) ve aynı anda hazır olmayabilecek Pod sayısı (maxUnavailable). İkincisi sıfıra ayarlanırsa kapasite hiç düşmez. Rolling update genelde kesintisizdir, ama geçiş sırasında eski ve yeni sürüm birlikte çalışır; API ve veritabanı değişikliklerinin geriye uyumlu olması gerekir. Rolling update'in güvenliği readiness probe'un doğruluğuna bağlıdır; readiness yoksa Kubernetes henüz hazır olmayan Pod'lara trafik gönderir.

**Blue-green** stratejisinde iki tam ortam hazır tutulur. Mavi ortam, yani eski sürüm, trafik alırken yeşil ortam, yani yeni sürüm, hazırlanır ve test edilir; hazır olduğunda trafik tek hamlede yeşile çevrilir. Geri dönüş aynı hızla yapılır. Bedeli, geçiş sırasında iki kat kapasite gerekmesidir.

**Canary** stratejisinde yeni sürüm önce trafiğin küçük bir kısmına açılır; hata oranı ve gecikme iyiyse oran kademeli olarak artırılır. Riski en çok azaltan stratejidir, ama güçlü bir gözlemlenebilirlik ve ağırlıklı yönlendirme gerektirir.

Kubernetes'in Deployment nesnesi yalnızca recreate ve rolling update'i bilir. Blue-green, iki Deployment ve Service selector'ının değiştirilmesiyle; canary, Gateway API'nin ağırlıklı yönlendirmesiyle; her ikisi de Argo Rollouts gibi bir progressive delivery aracıyla uygulanabilir. Bu araçlar adımları ve metrik analizini otomatikleştirir ve metrikler kötüyse kendiliğinden geri alır. Hangi yol seçilirse seçilsin, karar metriklere bağlanmalıdır.

Geri alma, eski revision'daki Pod şablonuna dönmek demektir; Kubernetes eski ReplicaSet'i yeniden büyütür. Ama veritabanı şeması gibi dış durum değişiklikleri geri alınmaz; yeni sürüm bir sütunu sildiyse eski sürüme dönmek o sütunu geri getirmez. Bu yüzden şema değişiklikleri geriye uyumlu ve birkaç adımda yapılır.

## Zamanlama ve güvenlik

Pod'ların yerleşimi etkilenebilir. **nodeSelector**, node etiketiyle birebir eşleşme ister; **node affinity** zorunlu kurallar ve tercihler tanımlamaya izin verir. Scheduler önce uygun olmayan node'ları eler, sonra kalanları puanlar. **Taint** ve **toleration** ise ters yönde çalışır: bir node'a konan taint Pod'ları iter ve yalnızca o taint'e toleration'ı olan Pod'lar oraya yerleşebilir. Pahalı GPU node'larını yalnızca GPU isteyen iş yüklerine ayırmak buna örnektir.

Güvenlikte üç kavram ayrılır. **Authentication** "kimsin?" sorusudur; Kubernetes'te kullanıcı nesnesi yoktur ve kullanıcılar OIDC, sertifikalar ya da bulut IAM'i gibi dış sistemlerle doğrulanır. **Authorization** "ne yapabilirsin?" sorusudur ve Kubernetes bunun için **RBAC** kullanır. **ServiceAccount** ise insanlar için değil, Pod'ların API ile konuşabilmesi için oluşturulan uygulama kimlikleridir.

RBAC dört nesneden oluşur. **Role** ve **ClusterRole** hangi API fiillerine ve kaynaklarına izin verildiğini tanımlar; ilki bir namespace içinde, ikincisi cluster genelinde geçerlidir. **RoleBinding** ve **ClusterRoleBinding** bir kullanıcıyı, grubu ya da ServiceAccount'u bir role bağlar. Role "ne yapılabilir", binding "bunu kim yapabilir" sorusunun cevabıdır. Kurallar `kubectl auth can-i` ile uygulanmadan önce ve sonra doğrulanabilir. En sık yapılan yetki hatası, kolaylık olsun diye cluster-admin vermektir.

Varsayılan olarak Kubernetes'te her Pod her Pod'a erişebilir. **NetworkPolicy**, Pod'ların giriş ve çıkış trafiğini etiketler, namespace'ler ve IP blokları üzerinden kısıtlar. Doğru yaklaşım önce her şeyi kapatmak, sonra yalnızca gereken yolları açmaktır: ön yüzden API'ye, API'den veritabanına. NetworkPolicy'yi CNI uygular; CNI desteklemiyorsa kurallar sessizce etkisiz kalır. Çıkış trafiği de kapatılırsa DNS için izin vermek unutulmamalıdır.

Pod güvenliği **securityContext** ile sağlanır: root olmayan kullanıcı, seccomp profili, yetki yükseltmenin engellenmesi, salt okunur kök dosya sistemi ve bütün capability'lerin kaldırılması. Namespace seviyesinde **Pod Security Standards** bunu zorunlu kılar: kısıtlamasız privileged, bilinen yetki yükseltmelerini engelleyen baseline ve root, capability ve seccomp kurallarını zorunlu kılan restricted seviyeleri vardır. Daha özel kurallar için Kyverno ya da OPA Gatekeeper gibi politika motorları kullanılır.

## Helm ve Headlamp

Bir uygulamayı çalıştırmak için Deployment, Service, ConfigMap, Secret ve Ingress gibi birçok manifest gerekir. **Helm**, Kubernetes için bir paket yöneticisidir ve bu manifest'leri bir **chart** içinde paketler.

Chart ile values birleşir, Helm bunlardan manifest'ler üretir ve Kubernetes API'sine uygular; bu kuruluma **release** denir. Her install, upgrade ya da rollback bir **revision** oluşturur. Üç kavram karıştırılmamalıdır: chart şablon paketidir, release chart'ın cluster'daki bir kurulumudur, revision ise o kurulumun her değişikliğidir. Aynı chart aynı cluster'a farklı release adlarıyla birden fazla kez kurulabilir. Helm bir Kubernetes kaynak türü değildir; yalnızca manifest üreten ve API'ye uygulayan bir araçtır.

Bir chart'ın yapısında paket bilgilerini ve bağımlılıkları taşıyan Chart.yaml, varsayılan yapılandırmayı tanımlayan values.yaml, manifest şablonlarını içeren templates dizini ve ortak şablon parçalarını tutan yardımcı dosyalar bulunur. Pipeline'larda `helm upgrade --install` sık kullanılır, çünkü release yoksa kurar, varsa günceller. Rollback da yeni bir revision olarak kaydedilir; geçmiş hiç silinmez.

Kubernetes Dashboard yıllarca cluster'a görsel bir arayüz olarak kullanıldı, ancak arşivlendi ve yerini **Headlamp** aldı. Headlamp, Kubernetes topluluğunun açık kaynak projesidir; kullanıcının kubeconfig'iyle çalışan bir masaüstü uygulaması ya da cluster içinde çalışan bir web uygulaması olarak kullanılabilir, birden çok cluster'ı tek arayüzden yönetir, kaynaklar arasındaki ilişkileri görsel olarak gösterir ve eklentilerle genişletilebilir. En önemli özelliği ek yetki vermemesidir: kullanıcının ne görüp ne yapabileceğini Kubernetes RBAC belirler.

> Kubernetes istenen durumu sürekli uzlaştıran bir kontrol sistemidir. Deployment, ReplicaSet ve Pod zinciri uygulamaları, Service ve DNS servis keşfini, PVC kalıcı depolamayı, probe'lar sağlığı, HPA ölçeklemeyi yönetir. Dağıtım stratejileri riski yönetir; RBAC, NetworkPolicy ve Pod güvenliği ise cluster'ı korur.
