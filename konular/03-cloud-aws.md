# Cloud computing ve AWS

Bulut, DevOps'un en büyük kolaylaştırıcılarından biridir. Bir test ortamı için haftalarca sunucu beklemek yerine birkaç dakikada ortam açıp iş bitince silebilmek, altyapıyı kodla tanımlamayı ve otomatikleştirmeyi mümkün kılar. Bu dokümanda bulutun mantığını, servis ve dağıtım modellerini, AWS'nin global altyapısını ve bir DevOps mühendisinin en sık kullandığı temel servisleri ele alıyoruz.

## Bulut nedir?

Bulutun en net tanımı şudur: kaynakları bir API üzerinden, talep üzerine ve ölçülebilir biçimde sunan bir işletim modeli. Geleneksel veri merkezinde kapasite önceden satın alınır, kurulum haftalar sürer, kapasite sabittir ve donanımı kurumun kendisi işletir. Bulutta ise paylaşılan bir kaynak havuzu vardır; ona bir API ile erişilir, talep edildiğinde kaynak gelir, yük artınca hızla ölçeklenir ve kullanılan kadar faturalandırılır.

DevOps açısından bu tanımdaki en önemli kelime API'dir. Altyapı bir API çağrısına dönüştüğü an otomatikleştirilebilir hâle gelir. Test ortamı dakikalar içinde açılıp kapatılabilir, altyapı Terraform gibi araçlarla Git'te tanımlanabilir, yük artınca kaynaklar otomatik büyüyüp azalabilir, veritabanı ve kuyruk gibi bileşenlerin işletimi sağlayıcıya devredilebilir. Ama bulut DevOps'u garanti etmez: arayüzde tıklanarak kurulan bir bulut ortamı, eski veri merkezinden pek farklı değildir.

## Dağıtım ve servis modelleri

Dört dağıtım modeli vardır. **Public cloud**'da sağlayıcının kaynakları hesap ve erişim politikalarıyla kullanılır. **Private cloud**'da bulut ortamı tek bir kuruluşa ayrılmıştır. **Hybrid cloud**'da kurum içi ortam ile public cloud birlikte çalışır; regülasyon nedeniyle verisinin bir kısmını kendi veri merkezinde tutması gereken kurumlarda yaygındır. **Multi-cloud**'da ise birden fazla sağlayıcı kullanılır.

Servis modelleri "kim neyi yönetir?" sorusuyla anlaşılır. Tamamen kurum içinde (on-premises) ağdan uygulamaya kadar her şeyi kurum yönetir. **IaaS** modelinde, örneğin AWS EC2'de, sağlayıcı donanımı, ağı ve sanallaştırmayı yönetir; işletim sistemi ve üstü müşteridedir. **PaaS** modelinde sağlayıcı runtime'a kadar her şeyi yönetir; müşteriye uygulama ve veri kalır. **SaaS** modelinde uygulamanın kendisi de sağlayıcıdadır; müşteri yalnızca kullanır ve erişim ayarlarını yapar. Sağa doğru gidildikçe operasyon yükü azalır, ama kontrol de azalır.

## Paylaşılan sorumluluk modeli

Bulut güvenliği sağlayıcı ile müşteri arasında paylaşılan bir sorumluluktur. AWS bunu şöyle özetler: AWS bulutun güvenliğinden, müşteri buluttaki her şeyin güvenliğinden sorumludur.

AWS fiziksel veri merkezlerini, donanımı, ağ altyapısını ve sanallaştırma katmanını korur. Müşteri ise verisini, IAM kullanıcılarını ve izinlerini, EC2 üzerindeki işletim sistemi yamalarını, güvenlik gruplarını, şifrelemeyi ve uygulama kodunu korur. Servis yönetildikçe sorumluluk sağlayıcıya kayar: EC2'de işletim sistemi yaması müşterideyken RDS ve Lambda'da sağlayıcıdadır. Ama veri ve erişim izni her durumda müşteride kalır. Bulutta yaşanan veri sızıntılarının büyük kısmı da sağlayıcının değil, müşteri tarafındaki hataların sonucudur: herkese açık bırakılmış depolama alanları, gereğinden geniş izinler, tüm internete açık yönetim portları.

## AWS global altyapısı

AWS'nin altyapısı üç kavramla anlatılır. **Region**, Frankfurt ya da İrlanda gibi coğrafi bir bölgedir. Region seçimi verinin nerede tutulacağını, yani veri yerleşimi regülasyonlarını, kullanıcıya olan gecikmeyi, hangi servislerin mevcut olduğunu ve felaket kurtarma stratejisini etkiler. **Availability Zone** (AZ), bir region içindeki ayrı bir arıza alanıdır: kendi elektriği, soğutması ve ağı olan bir ya da birden fazla veri merkezi. Bir region'da genellikle üç AZ bulunur; bunlar bir felaketin hepsini birden etkilemeyeceği kadar uzakta, ama aralarında senkron çoğaltma yapılabilecek kadar düşük gecikmeli bir ağla bağlıdır. **Edge** lokasyonları ise CloudFront ve Route 53 gibi global servislerin kullanıcıya yakın noktalarıdır.

Yüksek erişilebilirlik tasarımında ilk karar servis seçimi değil, arıza alanı seçimidir. Üretim sistemleri en az iki AZ'ye yayılır; böylece tek bir veri merkezinin çökmesi uygulamayı durdurmaz.

## AWS'ye erişim ve maliyet

AWS'ye aynı API'ye açılan dört kapıdan erişilir: keşif ve öğrenme için web arayüzü (Console), script ve hızlı kontrol için komut satırı aracı (CLI), uygulama içinden kullanım için SDK'lar ve ortamın tamamını kodla tanımlamak için altyapı kodu araçları. DevOps'ta tercih tekrarlanabilir olanlardır; arayüzde tıklanarak yapılan iş belgelenmez ve tekrarlanamaz.

Hesap güvenliğinin üç temel kuralı vardır: root kullanıcı yalnızca hesap kurulumunda kullanılır, çok faktörlü doğrulama açılır ve root için erişim anahtarı oluşturulmaz. İnsanlar için tek oturum açma ve geçici kimlik sağlayan IAM Identity Center tercih edilir. Geliştirme, test ve üretim ortamları için AWS Organizations ile ayrı hesaplar kullanılır.

Bulutta her şeyin bir saat ücreti vardır. Taahhütsüz, saniye ya da saat başı ödenen **on-demand** model deneme ve değişken yük için uygundur. Bir ile üç yıl taahhütle yüksek indirim sağlayan **Savings Plans** ve rezervasyonlar sürekli çalışan yük içindir. Boştaki kapasiteyi çok ucuza sunan ama geri alınabilen **spot** model kesintiye dayanıklı işler, örneğin CI agent'ları içindir. Üç alışkanlık maliyeti kontrol altında tutar: hesap açılır açılmaz bir bütçe alarmı kurmak, kaynakları proje, sahip ve ortam etiketleriyle işaretlemek ve iş bitince kaynakları silmek. Maliyet bir mühendislik metriğidir ve mimari kararların içinde konuşulmalıdır; bu yaklaşıma FinOps denir.

## IAM: kimlik ve erişim yönetimi

IAM (Identity and Access Management), kimlerin hangi kaynaklarda hangi işlemleri yapabileceğini yönetir ve region'dan bağımsız, global bir servistir.

Temel kavramlar şunlardır: **user** bir kişi ya da uygulama kimliği, **group** kullanıcı grubu, **policy** izinleri tanımlayan JSON belgesi, **role** geçici olarak üstlenilen kimlik ve **MFA** çok faktörlü doğrulama. Bir istek geldiğinde kimlik doğrulanır, ilgili policy'ler değerlendirilir ve izin ya da ret kararı verilir.

IAM'de en sık yapılan hata "çalışsın diye" verilen geniş izindir. İyi pratikler şunlardır: izinleri kişilere değil gruplara ya da izin setlerine vermek; uygulamaların ve pipeline'ların uzun ömürlü erişim anahtarları yerine **rol** üstlenmesini sağlamak; CI sistemlerinin buluta kısa ömürlü token'larla, OIDC üzerinden erişmesini sağlamak; yalnızca gereken eylem ve kaynaklara izin vermek, yani **en az yetki** ilkesini uygulamak ve kimin neye eriştiğini denetim kayıtlarıyla izlemek. İyi bir policy dardır: örneğin yalnızca belirli bir depolama alanında okuma ve yazma izni verir. Akılda tutulacak cümle şudur: bir sızıntının etkisi, sızan kimliğin yetkisi kadardır.

## VPC ve ağ güvenliği

VPC (Virtual Private Cloud), AWS içinde müşteriye ait, mantıksal olarak izole bir ağdır. Bir IP bloğu ile tanımlanır ve **subnet**'lere bölünür. **Route table**'lar trafiğin nereye gideceğini, **internet gateway** VPC'nin internete açılan kapısını, **NAT gateway** ise private subnet'teki makinelerin dışarı çıkabilmesini sağlar.

İki tür subnet vardır. **Public subnet** internetten doğrudan erişilebilir ve load balancer gibi bileşenler buraya konur. **Private subnet** internetten doğrudan erişilemez; uygulama sunucuları ve veritabanları burada durur. Temel kural şudur: dışarıdan erişilmesi gerekmeyen her şey private subnet'e.

Ağ trafiği iki katmanda kontrol edilir. **Security group** instance seviyesinde çalışır ve stateful'dur: içeri izin verilen trafiğin dönüşü otomatik olarak izinlidir. Yalnızca izin kuralı içerir ve kuralların hepsi birlikte değerlendirilir; başlangıçta gelen trafiğin tamamı kapalıdır. **Network ACL** ise subnet seviyesinde çalışır ve stateless'tır: dönüş trafiği için ayrı kural gerekir. İzin ve ret kuralları içerir ve kurallar numara sırasıyla değerlendirilir. İyi bir pratik, kuralları IP adreslerine değil diğer security group'lara referansla yazmaktır: "uygulama sunucusunun 8080 portuna yalnızca web sunucusu grubundan erişilebilsin." Böylece IP'ler değişse bile kural geçerli kalır. Kaçınılması gereken şey, yönetim portlarını tüm internete açmaktır.

## Temel servisler

**EC2** (Elastic Compute Cloud) bulutta sanal sunucu sunar. Bir instance başlatılırken beş karar verilir: işletim sistemi şablonu (AMI), CPU ve bellek kapasitesini belirleyen instance tipi, SSH için anahtar çifti, hangi portların kimden erişilebileceğini belirleyen security group ve ilk açılışta çalışan script (user data). Depolama tarafında **EBS** kalıcı bir blok disktir ve instance durduğunda da kalır; instance store ise geçicidir ve instance durunca silinir. Elle başlatılan her instance birbirinden biraz farklı bir "kar tanesi" olma eğilimindedir; tekrar kurulabilir olması için user data ya da altyapı kodu gerekir.

**Lambda**, sunucu yönetmeden kod çalıştırmayı sağlayan serverless bir servistir. Olay tabanlı çalışır, otomatik ölçeklenir ve yalnızca kodun çalıştığı süre için ücretlendirilir. Örneğin depolama alanına bir dosya yüklendiğinde tetiklenip onu işleyebilir. Kısa süreli ve olay odaklı işler için idealdir.

**SQS** (Simple Queue Service), uygulama bileşenleri arasında gevşek bağlı mesajlaşma sağlar. Bir servis mesajı kuyruğa koyar, diğeri kendi hızında alır. Alıcı servis kısa süre çökse bile mesajlar kaybolmaz; kuyruk yoğunluğu tamponlar. Defalarca işlenemeyen mesajları ayıran dead letter queue, bozuk bir mesajın bütün kuyruğu tıkamasını engeller.

**S3** (Simple Storage Service) nesne tabanlı depolama sunar ve yedekleme, arşivleme, statik içerik ve veri saklama için kullanılır. Temel kavramları global olarak benzersiz isimli **bucket**, dosya ve metadata'sından oluşan **object**, maliyet ve erişim sıklığına göre seçilen **storage class**, sürüm geçmişini tutan **versioning** ve erişim kurallarını belirleyen **bucket policy**'dir. Yanlışlıkla herkese açık bırakılmış bucket'lar bulut sızıntılarının en bilinen nedenlerinden biridir. S3, Terraform state'inin uzakta saklanması için de yaygın olarak kullanılır.

**RDS** (Relational Database Service), MySQL ve PostgreSQL gibi ilişkisel veritabanlarını yönetilen hizmet olarak sunar. Kurulum, yedekleme ve yamalama işlerinin büyük kısmını AWS üstlenir. **Multi-AZ** seçeneği başka bir AZ'de senkron bir kopya tutar ve ana veritabanı çökerse otomatik olarak ona geçer. Ağ izolasyonu, security group ve yedekleme stratejisi ise müşterinin kararıdır; veritabanı her zaman private subnet'te durmalıdır.

**CloudWatch** AWS kaynaklarından metrik, log ve olay toplar, eşik aşıldığında alarm üretir. Alarmlar bildirim gönderebilir, otomatik ölçeklemeyi ya da bir Lambda fonksiyonunu tetikleyebilir. EC2'nin temel metrikleri varsayılan olarak gelir; uygulama logları için ise bir agent kurmak gerekir.

## Servisler birlikte: üç katmanlı mimari

Bu servislerin bir mimaride nasıl birleştiğini klasik üç katmanlı yapı gösterir.

Kullanıcı DNS ve içerik dağıtım ağı üzerinden gelir. VPC içinde, public subnet'te bir Application Load Balancer bulunur. Private uygulama subnet'lerinde Auto Scaling grubuna bağlı EC2 instance'ları, private veritabanı subnet'inde RDS çalışır. Statik içerik ve yedekler S3'te, izleme ve alarmlar CloudWatch'ta durur. EC2 instance'ları S3'e erişim anahtarıyla değil, bir IAM rolüyle erişir. Bu mimaride internetten doğrudan erişilebilen tek bileşen load balancer'dır; uygulama ve veritabanı katmanları ancak bir üst katman üzerinden erişilebilir.

## Well-Architected Framework

AWS'nin Well-Architected Framework'ü iyi bir mimariyi altı sütun üzerinden değerlendirir. **Operasyonel mükemmellik**: değişikliklerin küçük ve geri alınabilir olması, operasyonun kodla yürütülmesi. **Güvenlik**: her katmanda kimlik, en az yetki ve izlenebilirlik. **Güvenilirlik**: hataya göre tasarlamak; birden çok AZ, otomatik kurtarma ve yedek. **Performans verimliliği**: doğru kaynak tipini seçmek, ölçmek ve ayarlamak. **Maliyet optimizasyonu**: ihtiyaç kadar kaynak, etiketleme ve bütçe. **Sürdürülebilirlik**: kullanılmayan kaynağı kapatmak ve verimli seçimler yapmak. Bu sütunlar birbirini dengeler; en ucuz mimari genellikle en güvenilir olan değildir.

AWS'nin kendi DevOps servisleri de vardır: Git deposu, yönetilen build ortamı, pipeline orkestrasyonu, dağıtım, container registry, container çalıştırma ve altyapı kodu servisleri. Bu dokümanlarda araçtan bağımsız kalmak için GitHub, GitLab, Jenkins ve Terraform'u kullanıyoruz; ama kavramlar aynıdır: kaynak, build, artifact ve dağıtım zinciri.

Bir son not: bulutu ezberlemek yerine kavramları tanımak önemlidir. Azure ve Google Cloud'da aynı kutular farklı isimlerle bulunur: VPC Azure'da VNet, EC2 Azure'da Virtual Machines ve Google'da Compute Engine, S3 Azure'da Blob Storage ve Google'da Cloud Storage olarak karşımıza çıkar.

> Bulut, altyapıyı bir API çağrısına çevirerek DevOps pratiklerini mümkün kılar. Güvenlik paylaşılan bir sorumluluktur; IAM'de en az yetki, ağda private subnet ve security group referansları, mimaride birden çok availability zone temel kurallardır.
