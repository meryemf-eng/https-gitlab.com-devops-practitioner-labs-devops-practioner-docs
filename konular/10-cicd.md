# CI/CD süreci

CI/CD, DevOps felsefesinin en somut pratiğidir: değişikliği kullanıcıya otomatik, güvenli ve tekrarlanabilir biçimde ulaştırmak. Bu doküman araçlardan bağımsız olarak CI/CD'nin kavramlarını, uçtan uca bir pipeline'ın aşamalarını ve pipeline tasarımını anlatır. Sonraki konularda aynı kavramları Jenkins, GitHub Actions ve GitLab CI ile uygulanmış hâliyle göreceğiz.

## CI/CD'den önce

CI/CD'nin değerini anlamak için onsuz bir teslimatı hatırlamak yeterlidir. Geliştirici "bitti" der ama kod haftalardır kendi branch'indedir. Operasyon ekibine bir iş talebi açılır ve kuyrukta bekler. Sunucular elle hazırlanır, dosyalar elle kopyalanır, yapılandırma elle düzenlenir. Sürüm gece yarısı, bir bakım penceresinde, bir dokümandaki adımlar izlenerek çıkarılır. Bir adım unutulur, geri dönüş planı yoktur ve suçlu aranır. Bu modelin sonuçları haftalar süren teslimat, yüzlerce değişikliğin aynı anda canlıya çıktığı büyük paketler, her seferinde farklı yapılan işler ve korkudur. Bir kısır döngü oluşur: acı veren iş seyrek yapılır, seyrek yapılan iş de hep acı verir.

Ortam farkları bu tabloyu kötüleştirir. Geliştiricinin bilgisayarı, test sunucusu ve üretim farklı işletim sistemleri, farklı runtime sürümleri, farklı kütüphaneler ve farklı ayarlar taşır. Ortamların zamanla elle yapılan değişikliklerle birbirinden uzaklaşmasına **configuration drift** denir. Çözümün üç ayağı vardır ve farklı konu dokümanlarında ele alınır: bir kez derlenip her ortamda kullanılan paket, uygulamayı bağımlılıklarıyla taşıyan container ve ortamları koddan aynı şekilde kuran altyapı kodu.

CI öncesinde entegrasyon da en sona bırakılırdı. Herkes kendi kodunu yazar, sonra her şey birleştirilir, derlenir ve test edilirdi. Testler ancak her şey bitince çalıştığı için geliştirici kodunun çalışıp çalışmadığını haftalar sonra öğrenir, bir test başarısız olduğunda binlerce satırlık değişikliğin içinde hatanın kaynağını arar ve sürümler giderek seyrekleşirdi. Buna "entegrasyon cehennemi" denir; entegrasyonu ertelemek riski biriktirmektir.

## Continuous integration, delivery ve deployment

**Continuous integration**, geliştiricilerin değişikliklerini günde birkaç kez ortak ana dala birleştirdiği ve her birleştirmenin otomatik build ve testlerle doğrulandığı pratiktir. Beş kuralı vardır: sık birleştirmek ve uzun yaşayan branch'lerden kaçınmak, her commit'te elle tetikleme olmadan build ve test çalıştırmak, build'i hızlı tutmak, kırık bir build'i her şeyin önüne koymak ve sonucu herkesin görmesini sağlamak. CI bir sunucu değil, bir alışkanlıktır; bir CI sunucusu kurup haftada bir birleştirme yapan ekip CI yapmıyordur. CI'ın özü şudur: her commit, otomatik olarak derlenen ve test edilen bir sürüm adayıdır.

**Continuous delivery**, aynı hattın üretime kadar uzanmasıdır, ama üretime geçişte bir insan onayı bulunur: her değişiklik üretime çıkmaya hazırdır, çıkış kararı ise bir iş kararı, bir bakım penceresi ya da bir regülasyon gereği olarak insana bırakılır. **Continuous deployment**'ta ise testleri geçen her değişiklik insan müdahalesi olmadan üretime çıkar.

İkisinde de pipeline aynıdır; continuous deployment, continuous delivery'nin güvenilir hâle gelmiş bir sonraki adımıdır. Her kurumun continuous deployment yapması gerekmez; her üretim değişikliği için bir onay kurulu zorunlu olan bir bankada doğru hedef continuous delivery'dir. Continuous deployment'ın ön koşulları güçlü bir test otomasyonu, otomatik geri alma ve canlı izlemedir.

## Uçtan uca pipeline

Bir dağıtım pipeline'ı beş büyük aşamada düşünülebilir: on dakikanın altında kalması gereken ve lint, birim testleri ve hızlı kontrollerden oluşan **commit** aşaması; bir kez çalışan ve bir artifact ya da container image üreten **build** aşaması; entegrasyon, güvenlik ve kabul testlerinin paralel çalıştığı **test** aşaması; üretime benzeyen bir ortamda doğrulama yapılan **staging** aşaması ve onaylı ya da otomatik **üretim** aşaması. Pipeline paralel aşamalarla hızlı sonuç verir, çalışmayan bir değişikliğin üretime ulaşmasını engeller ve her adımda bir denetim kaydı bırakır. Ama süreç ve kültür olmadan kimsenin kullanmadığı bir pipeline'a dönüşebilir ve kendisinin de bakıma ihtiyacı vardır. Temel kural şudur: pipeline üretime giden tek yoldur; elle bırakılan bir "kısa yol" onu anlamsız kılar.

Daha ayrıntılı bir haritada on iki aşama bulunur.

İlk beş aşama CI'dır. **Kaynak ve tetikleme** aşamasında geliştirici bir branch'e commit atar ve pull request açar; bir webhook pipeline'ı başlatır, branch protection ana dala doğrudan gönderimi engeller. Pipeline'ı push, pull request, tag, zamanlama ya da elle tetikleme başlatabilir. **Build** aşamasında bağımlılıklar lockfile'dan aynen kurulur ve kod temiz bir ortamda derlenir; build yalnızca pipeline'da geçerlidir. **Birim test** ve **statik analiz** aşamalarında kod kalitesi, bağımlılık açıkları ve depoya girmiş secret'lar denetlenir ve sonuç bir kalite kapısına bağlanır. **Paketleme** aşamasında uygulama bir container image'a ya da pakete dönüştürülür ve semantic version ile commit kimliğinden oluşan benzersiz bir sürüm alır.

Sonraki beş aşama continuous delivery'dir. **Registry** aşamasında paket bir registry'ye gönderilir; test, staging ve üretim aynı paketi kullanır ve ortamlar arasında yeniden build yapılmaz. **Test ortamına dağıtım** aşamasında paket, yapılandırması koddan gelen ve secret'ları güvenli bir depodan okunan bir ortama dağıtılır. **Entegrasyon ve uçtan uca test** aşamasında servisler arası API'ler, veritabanı ve kritik kullanıcı akışları sınanır. **Onay** aşamasında, continuous delivery'de, üretime çıkış kararı bir değişiklik kaydı ve sürüm notuyla birlikte verilir. **Üretim** aşamasında seçilen strateji yeni sürümü kademeli olarak devreye alır. Son iki aşama operasyon ve geri bildirimdir: **doğrulama** aşamasında sağlık kontrolleri ve metrikler izlenir ve eşik aşılırsa geri dönülür; **izleme** aşamasında metrik, log ve alarmlar canlı sistemi izler ve DORA metrikleri pipeline verisinden hesaplanır.

Her ok bir kalite kapısıdır: kapıyı geçemeyen değişiklik bir sonraki aşamaya ilerlemez.

### Testler ve kalite kapıları

Pipeline'daki testler dört kapı olarak düşünülebilir. **Hızlı kapı** build, lint, birim testleri, secret taraması ve statik analizden oluşur. **Sistem kapısı** entegrasyon, sözleşme ve API testlerinden oluşur. **Yayın kapısı** artifact imzalama, image taraması, dağıtım politikası ve onaydan oluşur. **Canlı doğrulama** ise smoke kontrolleri, hata oranı ve gecikme izlemesinden oluşur. Her test türü farklı bir riski yakalar ve tek bir uçtan uca test katmanı hızlı ve güvenilir geri bildirim için yeterli değildir. Kalite sonradan kontrol edilen bir şey değil, sürecin içine inşa edilen bir şeydir.

### Bir kez build, her yere aynı paket

CI/CD'nin en önemli ilkelerinden biri "build once, deploy many"dir: artifact bir kez üretilir ve değişmeden her ortama taşınır. Buna **promotion** denir. Kod aynı olsa bile her ortam için yeniden build yapılmaz, çünkü build anında bir bağımlılık güncellenmiş ya da bir base image değişmiş olabilir; o zaman test edilen paket ile üretime çıkan paket farklılaşır ve testin anlamı kalmaz. Ortamlar değişkendir, ama onaylanan artifact değişmez olmalıdır. Paketlerin etiketlenmesinde de aynı mantık geçerlidir: `latest` tek başına izlenebilir değildir; sürüm numarası ve commit kimliği, her image'ın hangi koddan üretildiğini gösterir.

### Deployment ve release

İki kavram sık karıştırılır. **Deployment**, yeni sürümün üretim ortamına dağıtılmasıdır ve teknik bir iştir. **Release** ise özelliğin kullanıcıya açılmasıdır ve bir ürün ve iş kararıdır. İkisini ayıran araç **feature flag**'dir: kod üretimdedir ama özellik bir bayrakla kapalıdır. Özellik önce küçük bir kullanıcı grubuna açılıp metrikler izlenebilir, sorun yoksa herkese açılabilir; sorun çıkarsa dağıtımı geri almadan yalnızca bayrağı kapatarak geri dönülebilir. Her değişiklik feature flag gerektirmez, ama riskli özelliklerde çok güçlü bir araçtır.

### Dağıtım ve geri alma

Üretime dağıtım, riski sınırlayan bir stratejiyle yapılır: kademeli güncelleme, blue-green, canary ya da feature flag. Bu stratejileri Kubernetes konusunde ayrıntılı ele aldık; özetle recreate kesintili ve basit, rolling kesintisiz ve varsayılan, blue-green iki kat kaynakla anında geri dönüşlü, canary riski küçük bir kullanıcı grubuyla sınırlayan stratejilerdir. Hangi strateji seçilirse seçilsin, veritabanı şeması geriye uyumlu değilse kesintisiz dağıtım yapılamaz.

Dağıtımdan hemen sonra sağlık kontrolleri, smoke testleri ve metrikler izlenir. Eşik aşılırsa önceki sürüme dönülür ya da düzeltme ileri taşınır. Geri alma ancak sürümlenmiş ve değişmez paketlerle saniyeler içinde yapılabilir; `latest` etiketiyle çalışan bir sistemde "önceki sürüm" diye bir şey yoktur.

## Pipeline tasarımı

**Branch modeli**, pipeline'ın ne zaman ve hangi ortama çalışacağını belirler. Trunk-based geliştirmede herkes kısa ömürlü branch'lerle günde birkaç kez ana dala birleşir; her commit'te build ve test çalışır, ana dal otomatik olarak staging'e gider ve sürümler tag ya da feature flag ile çıkar. GitHub Flow'da ana dal her zaman yayına hazırdır ve her iş bir branch ile pull request üzerinden ilerler. GitFlow'da develop, release ve hotfix branch'leri planlı sürümleri destekler ama uzun ömürlü branch riski taşır.

Tipik olarak dört **ortam** bulunur: her commit'in gittiği geliştirme, her birleştirmenin otomatik testlerden geçtiği test, sürüm adaylarının üretim kopyasında doğrulandığı staging ve onaylı sürümlerin kullanıcıya ulaştığı üretim. Aynı paket farklı ortamlarda yalnızca yapılandırmasıyla ayrışır; kod ile yapılandırmanın ayrılması Twelve-Factor ilkelerinden biridir. Parolalar ve token'lar pipeline'ın secret deposunda ya da Vault gibi bir kasada durur, depoda asla bulunmaz ve loglarda maskelenir.

Pipeline araçları farklı isimler kullanır ama kavramları aynıdır:

| Kavram | Jenkins | GitLab CI | GitHub Actions |
|---|---|---|---|
| Tanım dosyası | Jenkinsfile | .gitlab-ci.yml | .github/workflows/*.yml |
| Aşama | stage | stage | job |
| Çalıştırıcı | agent | runner | runner |
| Çıktı | archiveArtifacts | artifacts | upload-artifact |
| Sır | credentials | CI/CD variables | secrets |

Bir aracı iyi öğrenen, diğerlerini kavramları eşleyerek hızla öğrenir. Fark sözdizimindedir; düşünce aynıdır: tetikleyici, aşamalar, işler ve komutlar.

Pipeline'ı yöneten sunucu ile işi çalıştıran makine birbirinden ayrıdır.

Bir olay gelir, CI sunucusu işi kuyruğa koyar, bir runner ya da agent işi alıp çalıştırır ve sonuçlar geri döner. Platformun sağladığı **hosted runner**'larda kurulum gerekmez ve her iş temiz bir sanal makinede çalışır. **Self-hosted runner**'lar ise iç ağa ve özel donanıma erişebilir, ama bakımı ve güvenliği kuruma aittir; kurum içindeki bir cluster'a dağıtım yapılacaksa genellikle bu tür runner'lar gerekir. Pipeline'ı hızlandırmanın iki yolu, bağımlılıkları ve Docker katmanlarını önbelleğe almak ve testleri paralel işlere bölmektir.

## Push tabanlı CD ve GitOps

Dağıtımın kimin tarafından uygulandığı iki farklı model doğurur. **Push tabanlı** modelde pipeline, yani Jenkins, GitLab CI ya da GitHub Actions, kubectl ya da Helm ile değişikliği cluster'a iter. Bu modelde pipeline cluster'a erişim yetkisi taşır, dağıtım anında gerçekleşir ve sonrası izlenmez; birisi cluster'da elle bir şey değiştirirse fark edilmez.

**Pull tabanlı GitOps** modelinde ise Git istenen durumu tutar; cluster içinde çalışan bir ajan, örneğin Argo CD, Git'i izler, cluster ile karşılaştırır ve eşitler. Kimlik bilgisi cluster dışına çıkmaz, uzlaştırma süreklidir ve sapma görünür hâle gelip otomatik olarak düzeltilebilir. İki model rakip değildir: CI image'ı üretir ve Git'teki tanımı günceller, GitOps ajanı cluster'ı Git ile eşitler. GitOps'u ayrı bir konuda ele alacağız.

Araç seçimi ekosisteme göre yapılır. Jenkins bağımsız bir sunucudur; en büyük gücü esnekliği ve eklenti ekosistemi, en büyük yükü bakımıdır. GitLab CI deponun, merge request'in, registry'nin ve CI'ın tek bir platformda olmasını sağlar. GitHub Actions GitHub ile bütünleşiktir ve geniş bir hazır eylem pazarı sunar. Argo CD ise yalnızca dağıtım yapar, CI yapmaz. Kurumlarda sık görülen birleşimler GitLab CI ile Argo CD, GitHub Actions ile Argo CD ve Jenkins ile Argo CD'dir.

## Güvenlik ve ölçüm

Güvenlik pipeline'ın her aşamasındadır: commit'te secret taraması, build'de statik analiz ve bağımlılık taraması, image aşamasında image taraması, yazılım envanteri ve imza, dağıtımda politika kontrolü, çalışma anında izleme ve tepki. Bir açığı commit aşamasında bulmak dakikalar, üretimde bulmak günler ve itibar kaybı demektir.

CI/CD'nin başarısı araç sayısıyla değil, DORA metriklerindeki iyileşmeyle ölçülür. Sık ve küçük commit'ler dağıtım sıklığını, değişikliğin canlıya ulaşma süresini ve hata oranını iyileştirir. Otomatik testler ve kalite kapıları süreyi ve hata oranını, bir kez build ilkesi üçünü birden, otomatik dağıtım sıklığı, süreyi ve toparlanma süresini, kademeli yayın ve feature flag hata oranını ve toparlanma süresini, otomatik geri alma ve GitOps ise toparlanma süresini iyileştirir.

Kurumların CI/CD olgunluğu beş basamakta düşünülebilir: elle kopyalama ve dokümanlarla yapılan dağıtım, var olan ama elle çalıştırılan dağıtım script'leri, her commit'te build ve test yapan CI, her sürümün tek tıkla canlıya çıkabildiği continuous delivery ve otomatik çıkış, kademeli yayın ve otomatik geri almayla continuous deployment ile GitOps. Basamaklar atlanmaz; testsiz continuous deployment, hatayı otomatik olarak üretime taşır.

CI/CD'de sık yapılan hatalar da bellidir: geliştiricileri CI'dan uzaklaştıran yavaş pipeline'lar, bazen geçip bazen kırılan ve güveni yok eden kararsız testler, kırmızı build'e alışmak, her ortam için yeniden build etmek, secret'ları pipeline tanımlarında tutmak ve pipeline dışında yapılan, bir sonraki dağıtımda kaybolan elle düzeltmeler. Pipeline da bir üründür; hızı, güvenilirliği ve bakımı ölçülür.

> CI her commit'i otomatik olarak derler ve test eder; continuous delivery her sürümü hazır tutar, continuous deployment onu otomatik yayınlar. Pipeline üretime giden tek yoldur, artifact bir kez üretilip her ortama taşınır, deployment ile release feature flag ile ayrılır.
