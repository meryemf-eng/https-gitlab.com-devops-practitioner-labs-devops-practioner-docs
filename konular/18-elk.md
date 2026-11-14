# ELK ile log yönetimi

Önceki konuda metrikleri Prometheus ile topladık. Bu doküman üç sinyalin ikincisine, loglara odaklanıyor: dağınık log dosyalarından aranabilir, ilişkilendirilebilir ve alarm üreten bir log platformuna geçmek. Elastic Stack'in bileşenlerini, logların toplanması ve işlenmesini, Kibana ile aramayı, saklama ve güvenliği ve alternatif log platformlarını ele alıyoruz.

## Neden merkezi log?

Tek bir sunucuda logları `ssh` ile bağlanıp `grep` ile aramak işe yarar, ama bu yöntem ölçeklenmez. Kısa ömürlü Pod'lar silindiğinde logları da silinir. Bir sipariş altı servisten geçer ve logları altı farklı yerdedir. Trace ya da sipariş kimliğiyle bütün servislerde tek bir sorgu yapabilmek gerekir. Erişim ve değişiklik kayıtları denetim ve saklama yükümlülükleri nedeniyle tutulmalıdır. Belirli bir hata desenine göre alarm kurmak gerekir. Şüpheli girişler ve yetki yükseltmeleri gibi güvenlik olaylarının tespiti için de loglar merkezi olarak analiz edilmelidir. Kubernetes container loglarını standart çıktıdan alır ve node'da bir dizine yazar; merkezi bir toplayıcı oradan okur.

Bir logun yaşam döngüsü yedi adımdır: üretilir, toplanır, işlenir ve zenginleştirilir, saklanır ve indekslenir, aranır ve görselleştirilir, alarm üretir ve sonunda arşivlenir ya da silinir.

Her adımda bir karar vardır. Üretimde JSON biçimi ve ortak alan adları kullanılır; Elastic'in ortak şeması (ECS) bunun için bir standart sunar. Toplamada Filebeat, Elastic Agent ya da Fluent Bit gibi bir ajan seçilir. İşlemede loglar alanlara ayrılır, hassas veriler maskelenir ve bağlam eklenir. Saklamada ne kadar süre ve hangi katmanda tutulacağına karar verilir. Aramada kimin nasıl erişeceği belirlenir. Alarmda hangi desenin aksiyon gerektirdiği tanımlanır.

Log ve izleme ekosisteminde birçok araç vardır: en eski sistem izleme araçlarından Nagios, güçlü ama maliyetli kurumsal log platformu Splunk, AWS kaynakları için CloudWatch, merkezi log yönetimi için Elastic Stack, Elasticsearch'ün açık lisanslı bir çatalı olan OpenSearch ve etiket tabanlı, düşük maliyetli bir log deposu olan Grafana Loki.

## Elastic Stack

ELK, Elasticsearch, Logstash ve Kibana'nın baş harfleridir; toplayıcıların eklenmesiyle adı Elastic Stack oldu.

**Elasticsearch** dağıtık bir arama ve analiz motorudur; JSON dokümanlarını saklar, bir REST API sunar ve ters indeks ile hızlı arama yapar. **Logstash** girdi, filtre ve çıktı aşamalarından oluşan bir işleme hattıdır ve düz metin logları alanlara ayırır. **Kibana** arama, pano, alarm ve yönetim arayüzüdür. **Beats** ve **Elastic Agent** ise hafif toplayıcılardır. Basit kurulumlarda Logstash atlanır: toplayıcı doğrudan Elasticsearch'e gönderir ve işleme Elasticsearch'ün içindeki ingest pipeline ile yapılır. Elastic'in lisansı 2021'de değişti ve bu değişiklik OpenSearch çatalının doğmasına yol açtı; daha sonra açık kaynak bir lisans seçeneği yeniden eklendi.

Elasticsearch arama motoru kategorisinin en yaygın kullanılan sistemidir. Bu yüzden mapping, sorgu dilleri ve saklama yönetimi gibi kavramlar piyasanın en yaygın arama motorunda doğrudan kullanılır. Elasticsearch yalnızca loglar için değil, vektör ve hibrit arama ile yapay zekâ uygulamalarının altyapısında da kullanılmaktadır.

Elasticsearch kavramlarını veritabanı terimleriyle karşılaştırmak işi kolaylaştırır. **Cluster** ve **node** bir sunucu kümesine karşılık gelir ve node'lar master, data ve ingest gibi roller üstlenir. **Index** ya da **data stream** bir tabloya benzer; loglar için zaman tabanlı data stream'ler kullanılır. **Document** bir satıra benzer ve bir JSON belgesidir. **Mapping** bir şemaya benzer ve alan tiplerini tanımlar. **Shard** ve **replica** veriyi parçalara ve kopyalara bölerek paralellik ve dayanıklılık sağlar. **Inverted index** ise kelimelerden dokümanlara bir listedir ve tam metin aramayı hızlandırır.

Mapping'deki en önemli karar bir alanın **text** mi **keyword** mü olacağıdır. text tam metin arama içindir; bir analiz sürecinden geçer ve kelimelere bölünür; log mesajı text olur. keyword ise tam eşleşme, gruplama ve sıralama içindir; servis adı, log seviyesi ve trace kimliği keyword olur. Servis adı text olarak tanımlanırsa "payment-api" gibi bir ad parçalara bölünür ve servis başına gruplama yanlış sonuç verir. Tarih, sayı ve IP alanları için de özel tipler vardır. Elasticsearch alan tiplerini ilk gördüğü değerden tahmin edebilir, ama bu kontrolsüz alan artışına yol açabilir; doğrusu alan tiplerini bir index şablonuyla önceden tanımlamaktır.

## Toplama ve işleme

**Filebeat**, dosya ve container logları için hafif bir toplayıcıdır. Kubernetes'te her node'da bir DaemonSet olarak çalışır, container log dosyalarını okur, container biçimini çözer, JSON logları alanlara açar, Pod, namespace ve etiket gibi Kubernetes bilgilerini ekler ve gereksiz olayları düşürür. Elastic Agent ise Beats'in yerini alan ve merkezi olarak yönetilen tek bir ajandır.

**Logstash**'in hattı üç aşamadan oluşur: girdi, filtre ve çıktı. Filtrede **grok** düz metin satırlarını desenlerle alanlara böler, tarih filtresi zaman damgasını ayrıştırır, **mutate** alanları siler, yeniden adlandırır ya da maskeler. Logstash eski ya da düz metin loglar, karmaşık dönüşümler ve birden çok çıktı gerektiğinde kullanılır; JVM tabanlı olduğu için bellek ister ve basit durumlarda ingest pipeline yeterlidir. **Ingest pipeline** Elasticsearch'ün içinde dokümanı işler: mesajdaki JSON'u açar, zaman alanını ayrıştırır, parola gibi hassas alanları siler.

**Fluent Bit** ise C ile yazılmış, çok az bellek kullanan bir CNCF toplayıcısıdır ve Elasticsearch, Loki, OpenSearch ve nesne depolama gibi birçok hedefe gönderebilir. Toplayıcılar arasında seçim yapılırken bellek kullanımı, dönüşüm gücü ve hedef sistem dikkate alınır; OpenTelemetry Collector ise log, metrik ve trace'i birlikte toplar.

Kubernetes'te log toplamanın üç modeli vardır.

Önerilen model **node seviyesi ajandır**: her node'daki bir DaemonSet Pod'u o node'daki bütün container loglarını okur ve uygulamada değişiklik gerekmez. **Sidecar** modelinde dosyaya log yazan eski uygulamalar için aynı Pod'a ikinci bir container eklenir; kaynak maliyeti yüksektir. **Doğrudan gönderim** modelinde uygulama loglarını kendisi gönderir ve bağımlılık ile hata yönetimi uygulamaya geçer. Zenginleştirme alanları kritiktir: namespace ve Pod adı filtrelemenin temeli, image etiketi sorunu sürümle ilişkilendirmenin, trace kimliği ise logdan trace'e geçmenin yoludur.

## Kibana ile arama

Kibana'da log aramanın günlük aracı **Discover**, sorgu dili ise **KQL**'dir. Bir servisin hata seviyesindeki logları, belirli bir durum kodunun üzerindeki istekler, belirli bir trace kimliği ya da bir namespace'te belirli bir kelime geçen loglar kolayca filtrelenir. Hangi index'lere bakılacağını bir **data view** belirler. Her sorgu bir zaman aralığıyla çalışır; logların görünmemesinin en sık sebebi seçili zaman aralığıdır. Önce dar bir zaman aralığı ve keyword alanlarla başlamak gerekir; tam metin araması daha pahalıdır.

**ES|QL**, Elastic'in borulu sorgu dilidir; Linux'ta pipe ile komutları zincirlemeye benzer. Bir index'ten verileri alıp filtrelemek, gruplayıp saymak, sıralamak ve sınırlamak tek bir sorguda yapılır; örneğin son bir saatteki hata loglarını servis başına sayıp en çok hata verenleri listelemek. Kibana'da dört sorgu dili bulunur: Discover filtreleri için KQL, analiz ve panolar için ES|QL, API ve uygulama içi kullanım için Query DSL ve eski bir sözdizimi olan Lucene. **Lens** sürükle-bırak ile görselleştirme yapmayı, panolar bu görselleri bir araya getirmeyi sağlar. Panolar dışa aktarılıp Git'te tutulabilir.

Kibana'da log desenlerine göre **alarm** kurulabilir: örneğin ödeme servisinde son beş dakikada belirli sayının üzerinde zaman aşımı logu oluşursa ekip kanalına bildirim göndermek. Oran ve gecikme gibi koşullar için alarm metriklerden, belirli desenler ve olaylar için loglardan kurulur. Gürültüyü azaltmak için eşik, gruplama ve susturma kullanılır. Bazı bildirim entegrasyonları ve makine öğrenmesiyle anomali tespiti ücretli lisans gerektirir.

## İşletme

Loglar sürekli büyür. **ILM** (index lifecycle management), onları katmanlara taşır ve süresi dolanları siler.

**Hot** fazında veri hızlı diskte durur ve yazma burada yapılır. **Warm** fazında daha ucuz depolamaya taşınır ve salt okunur olur. **Cold** ve **frozen** fazlarında anlık görüntü tabanlı, çok ucuz depolamada tutulur. **Delete** fazında silinir. **Rollover** ile bir index belirli bir yaşa ya da boyuta ulaştığında yeni bir index'e geçilir. Shard'ların hedef boyutu onlarca gigabayt mertebesindedir; çok sayıda küçük shard cluster'ı yorar. Saklama süresi yasal zorunluluklar ile maliyet arasındaki bir dengedir. ILM olmayan bir log cluster'ının diski bir gün mutlaka dolar.

Elasticsearch'ün güncel sürümlerinde güvenlik varsayılan olarak açıktır: node'lar arası ve HTTP trafiği TLS ile şifrelenir, kurulumda sertifikalar üretilir, kullanıcılar ve API anahtarları ile kimlik doğrulanır ve rollerle yetki verilir; örneğin geliştiriciler logları okuyabilir ama üretime yazamaz. Hassas veriler toplama sırasında maskelenir ve alan düzeyinde güvenlik de kullanılabilir. Test ortamları için tek node'lu bir Docker Compose kurulumu yeterlidir; Kubernetes'te ise Elasticsearch, Kibana ve toplayıcıları Kubernetes kaynakları olarak tanımlayan bir operatör (ECK) kullanılır. Elasticsearch'ün çalışabilmesi için host'ta bir çekirdek parametresinin artırılması gerekir.

Üç farklı log platformu yaklaşımı vardır. **Elastic Stack** her alanı tam metin olarak indeksler, güçlü arama, analiz ve güvenlik analizi sunar, ama kaynak ve lisans maliyeti yüksektir. **OpenSearch**, Elasticsearch'ün açık lisanslı bir çatalıdır ve bir bulut sağlayıcının yönetilen hizmeti olarak da sunulur. **Grafana Loki** ise içeriği indekslemez, yalnızca etiketleri indeksler ve içeriği sıkıştırarak saklar; çok düşük maliyetlidir ve Prometheus'a benzer bir etiket mantığı kullanır, ama tam metin araması yavaştır. Seçim ihtiyaca göre yapılır: güçlü arama ve güvenlik analizi, açık lisans ya da düşük maliyet.

Log platformlarında sık karşılaşılan sorunlar da bellidir. Kibana'da log görünmüyorsa toplayıcının kendi loglarına, çıktı bağlantısına, data view desenine ve zaman aralığına bakılır. Cluster sağlık durumu sarı ya da kırmızıysa atanmamış shard'lar ve disk eşikleri kontrol edilir. Disk dolduysa ILM'in çalışıp çalışmadığına ve index'lerin salt okunur kilide girip girmediğine bakılır. Aynı alan farklı index'lerde farklı tipte tanımlandıysa bir mapping çakışması oluşur ve index şablonuyla çözülür. Kaçınılması gerekenler ise her şeyi hata ayıklama seviyesinde loglamak, parola ve token'ları loga yazmak, ILM olmadan çalışmak ve binlerce küçük shard oluşturmaktır.

> Merkezi log, kısa ömürlü Pod'ların ve dağıtık isteklerin dünyasında arama, korelasyon ve denetimi mümkün kılar. Kubernetes'te loglar bir DaemonSet ile toplanır, ingest pipeline ya da Logstash ile işlenir, Elasticsearch'te saklanır ve Kibana ile aranır. Mapping doğru alan tiplerini, ILM ise sürdürülebilir saklamayı sağlar.
