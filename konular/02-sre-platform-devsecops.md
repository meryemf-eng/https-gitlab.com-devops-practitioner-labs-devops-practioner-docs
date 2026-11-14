# SRE, platform engineering ve DevSecOps

DevOps bir felsefe ve çalışma biçimi olarak ne yapılması gerektiğini söyler, ama bunun nasıl yapılacağını her ayrıntısıyla tarif etmez. Zamanla bu boşluğu dolduran üç uzmanlık alanı doğdu: güvenilirliği ölçülebilir bir hedefe dönüştüren **SRE**, geliştiricilerin bilişsel yükünü azaltan **platform engineering** ve güvenliği teslimat akışının içine taşıyan **DevSecOps**. Bu doküman üçünün temel kavramlarını anlatır; araçlarını ilerleyen konularda göreceğiz.

## SRE nedir?

Site Reliability Engineering, Google'da 2003'te kuruldu. Kurucusu Ben Treynor Sloss onu şöyle tanımlar: SRE, bir yazılım mühendisinden operasyon ekibi tasarlamasını istediğinizde ortaya çıkan şeydir. Temel fikir, operasyon işlerini elle yapmak yerine yazılım mühendisliğiyle çözmek ve güvenilirliği bir umut olmaktan çıkarıp ölçülen ve yönetilen bir hedefe dönüştürmektir.

SRE ile DevOps'un ilişkisi bir yazılım benzetmesiyle anlatılır: SRE, DevOps arayüzünü uygulayan bir sınıftır. DevOps "siloları azalt" der; SRE'de geliştirme ve operasyon aynı araçları ve sorumluluğu paylaşır. DevOps "hatayı normal kabul et" der; SRE bunu hata bütçesi ve suçlamasız postmortem ile somutlaştırır. DevOps "küçük, kademeli değişiklik" der; SRE canary ve kademeli yayın kullanır. DevOps "otomasyonu kullan" der; SRE tekrarlayan operasyon işini ölçer ve sınırlar. DevOps "her şeyi ölç" der; SRE bunu kullanıcı odaklı göstergeler ve hedeflerle yapar.

Bir SRE mühendisinin işi altı başlıkta toplanabilir: ürünle birlikte güvenilirlik hedefleri koymak, aksiyon gerektiren alarmlar kurmak, olaylara müdahale etmek, tekrarlayan işi otomasyona çevirmek, kapasite planlamak ve olaylardan öğrenmeyi yönetmek. Google'ın SRE pratiğindeki önemli bir kural, bir SRE'nin zamanının en fazla yarısının operasyon işine gitmesidir; kalan zaman otomasyon ve iyileştirmeye ayrılır. Operasyon yükü bu sınırı aşarsa fazla iş geliştirme ekibine geri yönlendirilir. Bu kural, SRE ekibinin bir ticket çözme ekibine dönüşmesini engeller.

## SLI, SLO ve SLA

Güvenilirliği yönetmek için önce ölçmek gerekir. Bunun üç kavramı vardır.

**SLI** (service level indicator), ölçülen göstergedir; örneğin başarılı isteklerin tüm isteklere oranı. **SLO** (service level objective), bu gösterge için konulan hedeftir; örneğin "son 30 günde isteklerin en az yüzde 99,9'u başarılı olsun". **SLA** (service level agreement) ise müşteriyle yapılan, ihlal edildiğinde ceza ya da iade doğuran sözleşmedir. Pratikte iç SLO, SLA'dan biraz daha sıkı tutulur; böylece ekip sözleşme ihlal edilmeden önce uyarı alır.

İyi bir SLI kullanıcı yolculuğundan doğar. Bir e-ticaret sitesinde ürünlerin listelenmesi hızlı olmalı, sepete ekleme hata vermemeli, ödeme mutlaka başarılı olmalı, sipariş takibi güncel olmalıdır. Her biri için bir gösterge tanımlanabilir: erişilebilirlik (başarılı yanıt oranı), gecikme (belirli bir sürenin altında kalan isteklerin oranı), doğruluk, tazelik, kalite ve dayanıklılık. Formül her zaman aynıdır: iyi olay sayısı bölü geçerli olay sayısı.

Burada sık yapılan bir hatayı vurgulamak gerekir: sunucunun CPU kullanımı bir SLI değildir. Kullanıcı CPU'yu görmez; sayfanın açılıp açılmadığını ve ne kadar sürede açıldığını görür. CPU yüksekken kullanıcılar mutlu olabilir, düşükken sayfa hata veriyor olabilir. Ölçüm noktası da önemlidir: kullanıcıya ne kadar yakın ölçülürse o kadar doğru olur.

Hedeflerin ne anlama geldiğini kesinti süresiyle düşünmek faydalıdır:

| Erişilebilirlik | 30 günde izin verilen kesinti | Tipik kullanım |
|---|---|---|
| %99 | yaklaşık 7 saat 12 dakika | İç araçlar |
| %99,9 | yaklaşık 43 dakika | Çoğu web uygulaması |
| %99,95 | yaklaşık 22 dakika | E-ticaret ödeme |
| %99,99 | yaklaşık 4 dakika | Kritik platform |

Her ek "dokuz" yaklaşık on kat daha az kesinti, ama çok daha fazla maliyet demektir. Ayrıca güvenilirlik çarpılarak birleşir: yüzde 99,9 güvenilirliğe sahip üç servis seri bağlıysa toplam güvenilirlik yaklaşık yüzde 99,7'ye düşer. Bir servis, bağımlılıklarından daha güvenilir olamaz.

## Hata bütçesi

"Sistemimiz ne kadar güvenilir olmalı?" sorusuna verilen en yaygın cevap "yüzde 100"dür. SRE'nin cevabı ise hayırdır: yüzde 100 imkânsızdır, aşırı pahalıdır ve kullanıcı farkı zaten göremez, çünkü kendi telefonu ve ağı servisinizden daha az güvenilirdir.

**Hata bütçesi** bir eksi SLO'dur. SLO yüzde 99,9 ise bütçe yüzde 0,1'dir; bir milyon istekte bin hatalı istek hakkı vardır. Bütçe varsa ekip hızlı sürüm çıkarabilir, deney yapabilir, risk alabilir. Bütçe tükendiyse öncelik güvenilirlik işlerine geçer. Bu yaklaşım, geliştirmenin hız isteği ile operasyonun kararlılık arayışı arasında bir barış anlaşması işlevi görür: "dağıtım yapalım mı?" tartışması fikir kavgasından veriye taşınır. Şaşırtıcı ama önemli bir sonuç da şudur: hiç harcanmayan bir bütçe, ekibin gereğinden yavaş gittiğini gösterebilir.

Bir hesap örneği verelim. Ödeme servisinin SLO'su yüzde 99,9 ve 30 günde iki milyon istek geliyor; bütçe iki bin hatalı istektir. İlk hafta 300, ikinci hafta hatalı bir dağıtım yüzünden 1.100, üçüncü hafta 250 hata oluştuysa, dördüncü haftaya yalnızca 350 hatalı istek hakkıyla girilir. Bu durumda riskli sürümler bekletilir ve güvenilirlik işleri öne alınır.

Bu kararların kriz anında değil önceden yazılması gerekir; buna **hata bütçesi politikası** denir. Örneğin bütçenin yarısından fazlası kaldıysa normal sürüm çıkarılır, dörtte birin altına düştüyse yalnızca düşük riskli değişikliklere izin verilir, bütçe bittiyse acil düzeltmeler dışında değişiklik durdurulur, tek bir olay bütçenin beşte birini tükettiyse postmortem zorunlu olur. Eşikler kurumdan kuruma değişir; önemli olan politikayı ürün ve mühendislik ekiplerinin olaydan önce birlikte yazmasıdır.

## Toil

**Toil**, Türkçede angarya diyebileceğimiz, bir hizmeti çalıştırmakla ilgili ama kalıcı değer üretmeyen operasyonel iştir. Altı özelliği vardır: elle yapılır, tekrar tekrar gelir, otomatikleştirilebilir, bir kesinti ile tetiklenir, bittiğinde sistem iyileşmez ve trafikle doğrusal büyür. Her operasyon işi toil değildir; yeni bir arızayı araştırmak öğrenme sağlar ve kalıcı değer üretebilir.

Toil'un zararı sadece kaybedilen zaman değildir; kariyer gelişimini durdurur, morali düşürür ve hata riskini artırır. Amaç işi daha hızlı yapmak değil, o işi yapma ihtiyacını ortadan kaldırmaktır. Her gün aynı geçici hatayı elle düzeltmek yerine hatanın oluşma koşulunu kaldırmak, aynı erişim talebini tekrar tekrar işlemek yerine kurallı bir self-service mekanizması kurmak, disk dolunca logları elle silmek yerine bir saklama politikası tanımlamak ve sertifikayı her yıl elle yenilemek yerine otomatik yenileme kullanmak bu dönüşümün örnekleridir.

## Olay yönetimi

Bir olay beş aşamadan geçer: kullanıcı etkisini gösteren bir sinyalle algılanır, kapsamı ve önemi değerlendirilir, etkisi azaltılır, kalıcı olarak çözülür ve ondan öğrenilir.

Olay yönetiminin en önemli ilkesi **önce kanamayı durdurmaktır**. İlk amaç kök nedeni bulmak değil, kullanıcı etkisini sınırlamaktır: önceki sürüme dönmek, trafiği yönlendirmek ya da bir özellik bayrağını kapatmak. Kök neden sonra, sakin bir kafayla aranır. Bir başka ilke, her alarmın bir müdahale rehberine, yani runbook'a bağlı olmasıdır; rehberi olmayan bir alarm gecenin bir yarısında yalnızca panik üretir.

Büyük olaylarda roller net olmalıdır. **Incident commander** olayı yönetir, karar verir ve rolleri dağıtır; kendisi teknik müdahale yapmaz. **Operations lead** teknik müdahaleyi yürütür. **Communications lead** paydaşları düzenli bilgilendirir. **Scribe** zaman çizelgesini ve kararları kaydeder. Küçük olaylarda bir kişi birden fazla rol alabilir, ama olayı kimin yönettiği her zaman açıkça ilan edilir. Önem seviyeleri de önceden tanımlanmalıdır: ödemenin tamamen durması gibi bir olay en yüksek seviyede, kesintisiz müdahale gerektirir; bir iç raporun gecikmesi ise mesai içinde ele alınabilir.

Olay bittikten sonra yazılan postmortem bir şablon izler: özet, kullanıcı ve iş etkisi, dakika dakika zaman çizelgesi, katkıda bulunan etkenler, olayın nasıl fark edildiği, nelerin iyi ve kötü gittiği, sahipli ve tarihli aksiyonlar ve diğer ekiplerle paylaşılacak öğrenimler. Katkıda bulunan etkenler bölümünde tek bir kök neden aranmaz; olaylar genellikle koşulların birleşiminden doğar. "Ne iyi gitti" bölümüne şans faktörünün de yazılması önemlidir.

## Platform engineering

DevOps'un "you build it, you run it" ilkesi doğrudur, ama bir yan etkisi vardır. Bugün bir geliştiricinin Dockerfile, Kubernetes manifest'leri, Helm, Terraform, IAM ve secret yönetimi, CI pipeline'ı, kod kalitesi ve güvenlik taramaları, Prometheus, Grafana, log yönetimi, ağ, sertifikalar ve maliyet gibi konuların hepsine hâkim olması beklenir. İş değeri üretmesi gereken geliştirici zamanının büyük kısmını altyapıyla boğuşarak geçirir. Bu durum **bilişsel yük** olarak adlandırılır ve teslimatı yavaşlatır.

Platform engineering bu probleme verilen cevaptır: geliştiricilerin yazılımı güvenli ve kolay biçimde teslim etmesi için ortak bir platform sağlamak. Üç ilkesi vardır. Platformun müşterisi geliştiricinin kendisidir; platform ekibinin amacı işi devralmak değil, kolaylaştırmaktır. Platform self-service olmalıdır; ticket açıp günlerce sunucu beklemek yerine tek tıkla hazır, güvenli ve standart bir ortam alınabilmelidir. Ve platform bir ürün gibi yönetilir; kullanım kolaylığı, dokümantasyon ve kullanıcı geri bildirimi takip edilir.

CNCF'in tanımına göre platform, kullanıcıların ihtiyaçlarına göre seçilmiş ve birbirine entegre edilmiş yetenekler bütünüdür; ürün ekiplerinin uygulamalarını daha az koordinasyonla geliştirip işletmesini sağlar. Platform bir araç değil, araçları geliştiricinin kullanacağı hâle getiren bir üründür.

## Team Topologies

Matthew Skelton ve Manuel Pais'in Team Topologies kitabı, ekipleri akışa ve bilişsel yüke göre tasarlamak için dört ekip tipi önerir.

**Stream-aligned** ekipler bir iş akışına, örneğin sepet ve ödeme sürecine, uçtan uca sahip olur; organizasyonun büyük kısmı bu tipte olmalıdır. **Platform** ekipleri, stream-aligned ekiplerin hızlı ve bağımsız teslimatı için iç hizmetler sunar. **Enabling** ekipleri diğer ekiplere yeni yetenekler kazandırır ama işi onların yerine yapmaz; SRE ya da güvenlik koçluğu buna örnektir. **Complicated-subsystem** ekipleri ise derin uzmanlık gerektiren bir bileşeni, örneğin bir öneri motorunu, sahiplenir.

Ekipler arasında üç etkileşim modu tanımlanır. **Collaboration**'da iki ekip belirli bir süre yakın çalışır ve yeni bir şey keşfeder; bu mod geçicidir ve maliyetlidir. **X-as-a-Service**'te bir ekip diğerine hazır ve dokümante bir hizmet sunar; koordinasyon düşüktür ve ölçeklenir. **Facilitating**'de bir ekip diğerine yetenek kazandırıp çekilir. Hedef, platform ile ürün ekipleri arasındaki ilişkinin zamanla collaboration'dan X-as-a-Service'e evrilmesidir. Platformu kullanmak için her seferinde toplantı yapmak gerekiyorsa, platform henüz bir ürün olmamıştır.

## Internal Developer Platform ve golden path

Bir iç geliştirici platformu (IDP) katmanlardan oluşur. En üstte servis kataloğu, şablonlar ve dokümantasyon sunan bir **developer portal** vardır. Altında hazır servis iskeletleri, pipeline'lar ve Helm chart'larından oluşan **golden path** şablonları, onun altında CI/CD, GitOps ve registry'den oluşan **teslimat ve orkestrasyon** katmanı, en altta da Kubernetes, bulut ve ağdan oluşan **altyapı** bulunur. Bunları güvenlik ve uyum kuralları ile gözlemlenebilirlik yatay olarak keser.

Geliştirici en üst katmanla konuşur; alttaki karmaşıklık platform tarafından soyutlanır. Sonraki teknik konuların hemen hepsi bu katmanlardan birini kurar.

**Golden path**, bir işi yapmanın platform tarafından desteklenen, önerilen ve en kolay yoludur. Zorunlu değildir ama en kolay olandır. İçinde bir repo şablonu, hazır bir pipeline, Dockerfile, Helm chart, güvenlik varsayılanları, panolar ve alarmlar bulunur. Bu yoldan çıkmak serbesttir, ama o zaman sorumluluk ekibe geçer. İyi bir golden path, güvenli ve doğru olanı aynı zamanda en kolay olan yapar; insanlara kural dayatmak yerine doğru yolu en az dirençli yol hâline getirir.

Developer portal denince en çok anılan proje, Spotify'ın geliştirip açık kaynak yaptığı ve CNCF'e bağışladığı **Backstage**'tir. Backstage bir servis kataloğu, golden path şablonlarından tek tıkla yeni servis oluşturma, kodla birlikte yaşayan dokümantasyon ve eklentilerle diğer araçlara görünüm sunar. Yine de portal bir vitrindir; arkasında golden path ve otomasyon yoksa yalnızca bir web sitesidir.

## DevSecOps'a giriş

Geleneksel modelde güvenlik ekibi bir sürümü canlıya çıkmadan önce denetler. Bulgular geç gelir, düzeltme pahalıdır ve teslimat bekler. DevSecOps'ta ise güvenlik kontrolleri pipeline'ın içinde otomatik çalışır ve geliştirici bulguyu dakikalar içinde görür. Güvenlik sonda yapılan bir kontrol değil, yaşam döngüsünün her aşamasına gömülü bir özellik hâline gelir.

Bu yaklaşıma **shift-left**, yani sola kaydırma denir ve her aşamanın bir güvenlik pratiği vardır: planlamada tehdit modelleme, kodlamada secret taraması, build'de statik kod analizi ve bağımlılık taraması, testte dinamik test ve image taraması, release'de imza ve yazılım envanteri (SBOM), dağıtımda politika kontrolü, işletmede çalışma anı tespiti. Kazanç üç yönlüdür: maliyet düşer, çünkü kod yazılırken bulunan bir açık dakikalar içinde düzeltilir; hız artar, çünkü otomatik kontroller insan onayının yerine geçer; görünürlük artar, çünkü her bulgu pipeline çıktısında ve kod incelemesinde görünür.

Sorumluluk modeli de değişir: değişiklikten sorumlu olan, değişikliği yapan ekiptir; güvenlik ekibi rehber ve kural koyucudur. Pratikler ve araçlar şöyle eşleşir:

| Pratik | Hedeflediği risk | Araç örnekleri |
|---|---|---|
| SAST | Koddaki güvenlik açığı | SonarQube, Semgrep |
| SCA | Açık içeren kütüphane | Trivy, Dependabot |
| Secret scanning | Repoya giren parola ve token | Gitleaks |
| Container güvenliği | Image'daki açık | Trivy, Harbor |
| IaC taraması | Güvensiz altyapı tanımı | Trivy, Checkov |
| DAST | Çalışan uygulamadaki açık | OWASP ZAP |
| Supply chain | Kaynağı belirsiz paket | SBOM, imza |

DevSecOps'u araçlarıyla birlikte ayrı bir konuda ele alacağız.

## Rolleri birlikte düşünmek

DevOps mühendisi, SRE ve platform mühendisi birbirine yakın ama farklı odaklara sahip rollerdir. DevOps mühendisi teslimat hattına odaklanır ve "değişiklik hızlı ve güvenli gidiyor mu?" diye sorar; başarısını DORA metrikleriyle ölçer. SRE canlıdaki güvenilirliğe odaklanır ve "sistem yeterince güvenilir mi?" diye sorar; SLO, hata bütçesi ve toparlanma süresiyle ölçer. Platform mühendisi iç geliştiricilere odaklanır ve "ekipler bunu kendileri yapabiliyor mu?" diye sorar; platformun benimsenmesi ve geliştirici deneyimiyle ölçer. Sınırlar şirketten şirkete değişir ve küçük bir ekipte bu rollerin hepsini aynı kişi üstlenebilir.

Bir değişikliğin yolculuğu bu rollerin nasıl birlikte çalıştığını gösterir. Ürün ekibi golden path şablonuyla yeni bir servis oluşturur. Platformun sunduğu pipeline çalışır. Güvenlik kapıları devreye girer ve kritik bir açık varsa pipeline'ı durdurur. Açık düzeltilince değişiklik kademeli olarak dağıtılır. SRE'nin kurduğu alarm canary aşamasında hata oranındaki artışı yakalar, önceki sürüme dönülür ve postmortem'den çıkan aksiyonlar ilgili ekiplere dağılır. Platform yolu yapar, ekip sürer, güvenlik bariyer koyar, SRE yolu izler.

> SRE güvenilirliği SLO ve hata bütçesiyle ölçülebilir bir hedefe çevirir; platform engineering golden path ve self-service ile ekiplerin bilişsel yükünü azaltır; DevSecOps güvenliği yaşam döngüsünün başına alır ve otomatikleştirir. Üçü de aynı DevOps felsefesinin uzmanlık alanlarıdır.
