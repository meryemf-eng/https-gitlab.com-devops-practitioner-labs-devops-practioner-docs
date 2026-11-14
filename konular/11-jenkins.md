# Jenkins ile CI/CD

Jenkins, CI/CD dünyasının en eski ve hâlâ en yaygın araçlarından biridir. Birçok kurumun pipeline'ı onun üzerinde çalışır. Jenkins'in gücü esnekliğidir: neredeyse her şeyi yapabilir ve neredeyse her araçla konuşabilir. Aynı esneklik, bakımın tamamen kuruma ait olması anlamına gelir. Bu dokümanda Jenkins'in mimarisini, pipeline as code yaklaşımını, kaynak kod yönetimiyle entegrasyonunu, agent'ları ve güvenli işletimini ele alıyoruz.

## Jenkins nedir?

Jenkins, build, test ve dağıtım gibi yazılım teslim görevlerini otomatikleştiren açık kaynak bir otomasyon sunucusudur. Bir CI sunucusu olarak başlar ve binlerce topluluk eklentisiyle kaynak kod yönetimi, bulut, container, güvenlik ve raporlama araçlarına bağlanarak bir CD merkezine dönüşebilir. Pipeline tanımı Jenkinsfile ile versiyon kontrolüne taşınır.

Jenkins'in çözdüğü problem sürekli entegrasyonun kendisidir. Büyük ekiplerin geç entegrasyon, kırık build'ler ve uzun hata ayıklama süreleriyle boğuştuğu dönemlerde, günde en az bir entegrasyon ve her gece otomatik build ve test yapmak hataların çok daha erken bulunmasını sağladı. Bugün gece build'leri yerine her commit'te build hedeflenir; Jenkins bu fikri otomatikleştirir: commit, build, test, geri bildirim.

Test ve geliştirme ortamlarında Jenkins'i kurmanın en hızlı yolu Docker'dır. Uzun destek sürümü (LTS) image'ı, durumunun saklanacağı bir volume ile çalıştırılır; web arayüzü ve agent bağlantıları için iki port açılır. Üretimde LTS sürümü kullanılır, Jenkins bir reverse proxy ve TLS arkasına konur, düzenli yedek alınır ve controller üzerinde build çalıştırılmaz. Kurulum sırasında önerilen bütün eklentiler yerine ihtiyaç duyulanlarla başlamak bakım yükünü azaltır.

Jenkins'in bütün durumu **JENKINS_HOME** dizinindedir: job tanımları, genel ayarlar, credential'lar ve onları çözen gizli anahtarlar, build geçmişi, loglar, artifact'ler ve eklentiler. Bu dizin kaybedilirse Jenkins kaybedilir; bu yüzden kalıcı bir volume'da durmalı ve düzenli yedeklenmelidir. Credential'lar onları çözen anahtarlar olmadan geri yüklenemez; yedek bu anahtarları da kapsamalıdır.

## Mimari: controller ve agent'lar

Jenkins'in mimarisi bir controller ve agent'lardan oluşur.

**Controller** orkestrasyonu yapar: kaynak kod deposunu tarar, job yapılandırmalarını tutar, işleri kuyruğa alıp zamanlar, credential'ları ve eklentileri yönetir. **Agent**'lar asıl işi yapar: kodu çeker, derler, test eder, image üretir ve dağıtım komutlarını çalıştırır. Bir Linux ve Docker agent'ı, bir Windows agent'ı, bir Maven agent'ı ya da Kubernetes'te geçici olarak açılan bir Pod agent olabilir. Temel kural, build, test ve dağıtım işlerinin agent'larda çalışması, controller'da çalışmamasıdır.

Bir build'in yolculuğu dört kavramla anlatılır.

Build önce **kuyruğa** girer; uygun bir agent yoksa ya da eşzamanlılık kısıtı varsa bekler. Pipeline bir **label** ile ne tür bir agent istediğini söyler ve uygun agent eşleştirilir. **Executor**, bir agent üzerindeki paralel iş kapasitesidir. **Workspace** ise kodun ve build dosyalarının agent'ın diskinde durduğu yerdir. Çok executor her zaman daha hızlı değildir, çünkü aynı işlemci ve disk için yarışırlar. Geçici agent'larda workspace build bitince kaybolur; kalıcı olması gereken çıktılar artifact olarak saklanmalıdır.

Jenkins'te dört job türü vardır. **Freestyle** job'lar arayüzden tıklanarak yapılandırılır, pipeline as code değildir ve yalnızca basit görevler için uygundur. **Pipeline** job'ları bir Jenkinsfile ile tanımlanır. **Multibranch pipeline** depodaki her branch ve pull request için otomatik olarak bir pipeline oluşturur. **Organization folder** ise bir organizasyondaki bütün depoları ve branch'leri keşfeder.

Eklenti ekosistemi Jenkins'in hem gücü hem riskidir. Kaynak kod yönetimi, pipeline, bulut ve dağıtım için yüzlerce eklenti vardır. Ama her ihtiyaca bir eklenti yerine asgari bir eklenti seti kullanmak, güncellemeleri önce bir test controller'ında denemek ve bakımı bırakılmış eklentileri kritik yolda kullanmamak gerekir; Jenkins'teki güvenlik açıklarının önemli bir kısmı eklentilerden gelir.

## Pipeline as code

Pipeline as code yaklaşımında pipeline tanımı, yani **Jenkinsfile**, uygulama koduyla aynı depoda durur. Bunun üç kazancı vardır: pipeline değişikliği de pull request ve kod incelemesinden geçer, hangi commit'in hangi pipeline mantığıyla derlendiği izlenebilir ve arayüzde tıklanarak yapılan, zamanla kayan ayarlar yerine kod kullanılır.

Jenkinsfile iki sözdizimiyle yazılabilir. **Declarative** sözdizimi belirli bir yapıya sahiptir, okunabilirdir ve baştan doğrulanabilir; çoğu pipeline için varsayılan tercihtir. **Scripted** sözdizimi Groovy tabanlı serbest bir dildir; çok esnektir ama okunabilirliği yazanın deneyimine bağlıdır ve karmaşık özel mantık gerektiğinde kullanılır. Declarative bir pipeline içinde gerektiğinde küçük script blokları açılabilir.

Declarative bir Jenkinsfile'ın iskeleti dört parçadan oluşur. **agent** pipeline'ın nerede çalışacağını seçer. **stages** teslim sürecinin ana fazlarıdır. Her **stage** içindeki **steps** agent üzerinde çalışan gerçek komutlardır. **post** ise sonuca göre yapılacakları tanımlar: her durumda, yalnızca başarılı olursa ya da yalnızca başarısız olursa. agent pipeline seviyesinde verilebileceği gibi, `agent none` ile global bir executor tutulmadan her stage'in kendi agent'ını seçmesi de sağlanabilir; böylece araç zinciri, işletim sistemi ve güvenlik sınırı stage'e göre ayrılır.

Jenkinsfile'da birkaç blok daha sık kullanılır. **environment** ortam değişkenlerini, **tools** Jenkins'in yönettiği araç kurulumlarını, **options** ise politikaları tanımlar. Options içinde üç ayar her pipeline için önerilir: takılan bir build'i sonsuza kadar bekletmeyen zaman aşımı, aynı job'ın iki build'inin çakışmasını engelleyen eşzamanlılık kısıtı ve eski build'leri temizleyerek diskin dolmasını önleyen build geçmişi sınırı. **post** bloğunda test raporu yayınlama ve temizlik gibi her durumda yapılması gerekenler yalnızca başarılı yola konmamalıdır; test başarısız olduğunda rapor en çok o zaman gerekir.

İki girdi kavramı ayrılmalıdır. **parameters** build başlarken verilen değerlerdir; örneğin hangi ortama dağıtım yapılacağı. **input** ise pipeline çalışırken durup bekleyen bir onay kapısıdır; continuous delivery'deki manuel onay adımı budur. Onayı kimin verebileceği de tanımlanmalıdır. Bir ayrıntı önemlidir: onay, bir agent tutan stage içinde beklenirse, biri onaylayana kadar o executor kilitlenir; bu yüzden onay agent'ı olmayan bir stage'de beklenmelidir.

Birbirinden bağımsız testler **parallel** stage'lerle eşzamanlı çalıştırılarak toplam süre kısaltılabilir, ama bu executor kapasitesi gerektirir; kapasite yoksa paralel stage'ler kuyrukta bekler. Pipeline'ları görselleştirmek için eskiden kullanılan Blue Ocean'ın yerini, standart arayüz içinde sıralı ve paralel stage'leri tek grafikte gösteren Pipeline Graph View almıştır.

**Credential**'lar Jenkins'in kendi deposunda saklanır ve yalnızca ihtiyaç duyan adımda açılır. `withCredentials` bloğu bir kullanıcı adı ve parolayı ya da bir token'ı yalnızca o blok içinde ortam değişkeni olarak sunar. Türleri kullanıcı adı ve parola, secret metin, SSH anahtarı, secret dosya ve sertifikadır; kapsamları global, klasör ya da kullanıcı düzeyinde olabilir. Loglarda maskelenirler, ama değer base64 gibi bir dönüşümden geçirilirse maske atlanır. Kritik bir ayrıntı da shell komutlarının tek tırnakla yazılmasıdır: tek tırnakta değişkeni shell açar ve secret Groovy'nin metin işlemesine hiç girmez; çift tırnakta ise Groovy değeri komut satırına yerleştirir ve secret process listesine ya da loglara düşebilir.

Build çıktıları için üç araç vardır. `junit` test sonuçlarını yayınlar ve build sayfasında test eğilimini gösterir. `archiveArtifacts` çıktıyı build kaydına ekler ve hangi build'de üretildiğini izlenebilir kılar. `stash` ve `unstash` aynı çalışma içinde agent'lar arasında küçük dosyaları taşır. Büyük artifact'ler controller'a değil, Nexus ya da Harbor gibi bir depoya gönderilmelidir.

## Kaynak kod yönetimi ve multibranch

Multibranch pipeline'da kaynak kodu almak için tek bir komut yeterlidir: `checkout scm`. Bu komut, Jenkinsfile'ın geldiği branch'in ya da pull request'in tam olarak o commit'ini çeker. Branch adı elle yazılarak kod çekilirse, pipeline'ın test ettiği commit ile tetiklenen commit farklı olabilir.

Pipeline'ı tetiklemenin iki yolu vardır. **Webhook**'ta Git platformu bir olay olduğunda Jenkins'e haber verir; gecikme ve yük düşüktür. **Polling**'de ise Jenkins her birkaç dakikada bir "yeni bir şey var mı?" diye sorar; gecikme zamanlamaya bağlıdır ve gereksiz yük oluşur. CI için webhook tercih edilir; polling yalnızca Jenkins'e dışarıdan erişilemiyorsa kullanılır. Zamana bağlı gece işleri ve bakım görevleri için ise cron kullanılır.

Kaynak kod platformu ile Jenkins arasındaki sözleşme dört adımdır: bir olay gerçekleşir, gizli bir token ile imzalanmış bir webhook Jenkins'e gelir, Jenkins ilgili branch'i bulup olayın geldiği commit'i çeker ve pipeline'ı çalıştırır, sonuç platforma bir commit durumu olarak geri yazılır ve başarısız bir build birleştirmeyi engeller. Geliştirici sonucu Jenkins'e girmeden, pull request ekranında görür.

**Multibranch pipeline**, Jenkinsfile içeren bütün branch'leri ve pull request'leri otomatik keşfeder ve her biri için ayrı bir alt job oluşturur; branch silindiğinde job da kaldırılır.

Multibranch bağlamı pipeline mantığına da taşınabilir: bir stage yalnızca ana dalda, örneğin staging'e dağıtım için, ya da yalnızca pull request'lerde, örneğin doğrulama için çalıştırılabilir. Yine de üretim dağıtımı yalnızca "ana dalda mı?" kontrolüne bağlanmamalı, yetkilendirme ve ortam politikasıyla birlikte düşünülmelidir.

GitLab entegrasyonunda Jenkins'te bir GitLab bağlantısı ve API token'ı, GitLab'da ise Jenkins'in adresine ve gizli token'a sahip bir webhook tanımlanır; sonuç merge request'te görünür ve "pipeline başarılı olmalı" ayarıyla kırmızı build'de birleştirme kilitlenir. GitHub tarafında ise kişisel token yerine bir GitHub App önerilir; daha yüksek API sınırları ve depo bazında izin sağlar. Stage sonuçları pull request'te kontrol olarak görünür. Fork'tan gelen pull request'lere dikkat etmek gerekir: başkasının yazdığı bir Jenkinsfile'ı çalıştırmak, güvenilmeyen kodu kendi agent'ınızda çalıştırmak demektir; varsayılan olarak yalnızca yazma yetkisi olan kişilerin pipeline değişiklikleri güvenilir sayılır.

## Agent'lar, Docker ve Kubernetes

Label'lar bir niyet ifadesidir: "bu iş Linux ve Docker olan bir makine istiyor." Label tasarımında araç zinciri ya da güven sınırı ifade edilmeli, anlamsız isimlerden kaçınılmalıdır. "Waiting for next available executor" mesajı çoğu zaman bir kapasite ya da label sorununa işaret eder.

Docker Pipeline eklentisiyle stage'ler bir container image'ı içinde çalıştırılabilir; örneğin Maven testleri Maven ve Java içeren bir image'da koşar ve agent'a ayrıca Maven kurmak gerekmez. Araç zinciri image ile standartlaşır ve agent makinesi sade kalır. Ama Docker soketine erişimin güçlü bir yetki olduğu unutulmamalı ve önbellek stratejisi ayrıca tasarlanmalıdır.

Kubernetes eklentisi ise ihtiyaç olduğunda bir agent Pod'u oluşturur ve build bitince siler. Pod'un içinde agent container'ının yanında Maven ya da Docker CLI gibi araç container'ları bulunabilir ve her stage istediği container'da çalışır. Sabit agent makineleri olmaz, her build temiz bir ortamda başlar ve kapasite cluster kadar büyür.

## Güvenlik ve işletim

Jenkins controller'ı bütün secret'ları ve dağıtım yetkisini tuttuğu için saldırganlar için en değerli hedeflerden biridir. Altı kural öne çıkar: controller'daki yerleşik node'un executor sayısını sıfır yapmak ve build'lerin JENKINS_HOME'a erişmesini engellemek; anonim okumanın kapalı olduğu, klasör bazında takım yetkisi veren matris ya da rol tabanlı bir yetki modeli kullanmak; kimlik için LDAP ya da tek oturum açma kullanmak; güvenli alan dışındaki Groovy çağrılarına verilen onayları gözden geçirmek; güvenlik duyurularını izleyip çekirdek ve eklenti güncellemelerini düzenli uygulamak ve arayüzü TLS arkasına, agent portunu iç ağa almak. Jenkins'i ele geçiren, bütün credential'ları ve üretime dağıtım yetkisini ele geçirir.

Onlarca Jenkinsfile'da aynı adımları kopyalamak yerine bir **shared library** yazılabilir. Kütüphanede tanımlanan bir adım, örneğin bir Node.js CI akışı, uygulama deposundaki Jenkinsfile'dan tek satırla çağrılır. Kütüphanenin sürümü bir tag ile sabitlenmelidir; aksi hâlde kütüphanedeki bir değişiklik bütün pipeline'ları aynı anda kırabilir. Shared library, platform engineering'deki golden path fikrinin Jenkins'teki uygulamasıdır; GitHub'da reusable workflow, GitLab'da CI/CD components onun karşılığıdır. Jenkins'in kendi yapılandırması da **Configuration as Code** eklentisiyle koda dökülebilir.

Jenkins'te sorun giderirken önce sorun sınıflandırılır: kuyrukta mı, agent'ta mı, kaynak kod platformunda mı, script'te mi, yoksa hedef sistemde mi? Label eşleşmeyen işler kuyrukta bekler, gelmeyen webhook'lar için platformun teslim kaydına bakılır, script onayı gereken çağrılar onay ekranında görünür, çevrimdışı agent'lar için ağ ve Java sürümü kontrol edilir, açılmayan Pod agent'ları için Pod olaylarına bakılır, dolan disk için build geçmişi sınırlanır.

Jenkins anti-pattern'leri de bellidir: controller'da build çalıştırmak, arayüzden yapılandırma yapmak, her ihtiyaca bir eklenti kurmak, secret'ları çift tırnakla kullanmak, polling ile tetiklemek, güncellemeleri ertelemek, sınırsız build geçmişi tutmak ve onay beklerken executor kilitlemek. Jenkins güçlüdür, ama bakımı sizin işinizdir.

> Jenkins bir controller ve agent'lardan oluşur; build'ler agent'larda çalışır. Declarative Jenkinsfile pipeline'ı kod olarak tanımlar, multibranch her branch için pipeline'ı otomatikleştirir, webhook'lar tetiklemeyi anlık yapar. Güvenlik, eklenti disiplini ve yedek Jenkins işletiminin ayrılmaz parçalarıdır.
