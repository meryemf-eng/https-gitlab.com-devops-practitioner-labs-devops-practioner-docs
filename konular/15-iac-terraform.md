# Altyapı kodu: Terraform ve Ansible

Altyapı değişikliği kişilere ve arayüzlere bağlı kaldığında tekrar üretmek, incelemek ve güvenle sürdürmek zorlaşır. Aynı ortamı iki kişi farklı sırayla ve farklı ayrıntılarla kurar; kimin neyi neden değiştirdiği bulunamaz; test ve üretim zamanla birbirinden ayrışır. **Infrastructure as Code** (IaC), altyapıyı makine tarafından işlenebilir tanımlarla yönetmek ve bu tanımları versiyon kontrolünde tutmaktır. Bu dokümanda IaC yaklaşımlarını, Ansible'ın yerini ve Terraform'u ayrıntılı olarak ele alıyoruz.

## IaC yaklaşımları

IaC'nin değeri yalnızca kaynakları hızlı oluşturmak değildir. Aynı ağ ya da uygulama kalıbı farklı ortamlarda yeniden yazılmadan kullanılır. Değişiklik geçmişi, inceleme ve geri dönüş konuşulabilir hâle gelir. Değişikliğin etkisi önceden görülür ve onaylanan değişiklik hızlı ve tutarlı uygulanır. Aynı altyapı bilgisini kopyalamak yerine bir kez tanımlayıp güvenle tekrar kullanmak, DRY ilkesinin altyapıdaki karşılığıdır.

Araç seçmeden önce iki eksen düşünülür.

Birinci eksen tarif biçimidir. **Declarative** yaklaşımda olması gereken son durum yazılır; Terraform ve Kubernetes manifest'leri böyledir ve tekrar çalıştırıldığında fark yoksa hiçbir şey yapmaz. **Imperative** yaklaşımda yapılacak adımlar yazılır; shell script'leri böyledir ve tekrar çalıştırıldığında adımlar yeniden yapılır. İkinci eksen değişikliğin nasıl uygulandığıdır. **Mutable** yaklaşımda çalışan sunucuya yeni paket ya da ayar uygulanır; Ansible, Puppet ve Chef böyle çalışır ve zamanla sunucuların birbirinden ayrışması riski taşır. **Immutable** yaklaşımda yeni bir image ya da sanal makine üretilip eskisi atılır; container'lar ve Kubernetes bu yaklaşımın en yaygın örnekleridir. Pratikte ikisi birlikte kullanılır: değişmez bir image ve açılışta küçük bir yapılandırma.

Her IaC aracı farklı bir katmanda güçlüdür. Terraform, OpenTofu, Pulumi ve CloudFormation ağ, sanal makine, veritabanı ve depolama gibi kaynakları **oluşturur**. Packer ve Dockerfile **image** üretir. Ansible, Puppet ve Chef sunucu içindeki paketleri, dosyaları ve servisleri **yapılandırır**. Helm, Kustomize ve Argo CD **Kubernetes içindeki** kaynakları yönetir. OPA, Checkov ve Trivy ise altyapı kodunu **politikalara** göre tarar.

## Terraform ve Ansible

Terraform ile Ansible rakip değil, farklı odaklara sahip araçlardır. Terraform altyapının yaşam döngüsünü yönetir: VPC, subnet, sanal makine, depolama ve yönetilen servisleri oluşturur ve günceller. Ansible ise çalışan sunucunun yapılandırmasını yönetir: paketler, dosyalar, servisler ve uygulama ayarları. Tipik bir örnek, Terraform ile bir sanal makine oluşturup Ansible ile üzerine bir web sunucusu yapılandırmasıdır.

**Ansible**, kurulum, yapılandırma, dağıtım ve orkestrasyonu otomatikleştiren açık kaynak bir araçtır. En belirgin özelliği agent gerektirmemesidir: hedef makinede yalnızca SSH ve Python yeterlidir. Görevler okunması kolay YAML **playbook**'larında tanımlanır. Binlerce hazır modül **idempotent** çalışır, yani istenen durumu kontrol eder ve yalnızca gerekirse değişiklik yapar. Sunucular, bulut kaynakları, ağ cihazları ve Windows makineler yönetilebilir; şifreli değişkenler için Ansible Vault kullanılır.

Bir playbook hangi sunucularda, hangi görevlerin, hangi sırayla çalışacağını tanımlar. Bir **play** bir host grubu ve görev listesidir. Bir **task** bir modül çağrısıdır. Bir **handler** ise yalnızca bir task tarafından tetiklendiğinde ve play sonunda bir kez çalışır; örneğin yapılandırma dosyası değiştiğinde web sunucusunu yeniden başlatmak. Playbook'lar önce kuru çalıştırma moduyla, değişecekleri göstererek denenebilir.

İki araç birlikte şöyle çalışır: Terraform sanal makineleri oluşturur ve etiketler, Ansible'ın dinamik inventory'si bulut sağlayıcısına belirli etiketli makinelerin hangileri olduğunu sorar ve playbook bu makineleri yapılandırır. Aradaki köprü inventory'dir. Kubernetes kullanılan ortamlarda sunucu içi işlerin çoğu container'lara geçer ve Ansible yalnızca node hazırlığı yapar.

## Terraform'un mimarisi

Terraform'un çalışma modeli dört parçadan oluşur: HCL ile yazılan yapılandırma, onu işleyen Terraform core, platformların API'leriyle konuşan provider'lar ve yapılandırmadaki kaynaklarla gerçek kaynakların eşleşmesini tutan state.

**Terraform core** yapılandırma dosyalarını okur, kaynaklar arasındaki bağımlılık grafiğini kurar, istenen durum ile mevcut durum arasındaki farkı hesaplar ve provider çağrılarını doğru sırayla yapar. Core AWS'yi, Docker'ı ya da Kubernetes'i bilmez; bu iş **provider**'lara aittir. Provider, core'dan gelen isteği platformun API'sinin anlayacağı biçime çeviren bir eklentidir ve hangi kaynakların kullanılabileceğini belirler. Bu ayrım sayesinde Terraform yüzlerce farklı platformu aynı dille yönetir.

Terraform HashiCorp tarafından geliştirilir ve 2023'ten beri açık kaynak olmayan bir lisansla dağıtılmaktadır. Bunun üzerine topluluk, Linux Foundation çatısı altında açık kaynak bir çatal oluşturdu: **OpenTofu**. Komut adı farklıdır ama sözdizimi büyük ölçüde aynıdır; bu dokümandaki kavramlar ikisinde de geçerlidir. Hangisinin kullanılacağını kurumun lisans politikası belirler. Hangi araç kullanılırsa kullanılsın, Terraform ve provider sürümleri yapılandırmada sabitlenmelidir.

## HCL'nin yapı taşları

HCL, adım adım komut listesi değil, istenen altyapı durumunu tarif eden declarative bir dildir. Üç temel blok vardır. **terraform** bloğu Terraform ve provider sürüm kısıtlarını içerir; `required_providers` her provider'ın registry adresini ve uyumlu sürüm aralığını belirtir. `terraform init` sırasında seçilen provider sürümleri bir **lock dosyasına** kaydedilir ve bu dosya Git'e commit edilir; npm'deki lockfile'ın Terraform karşılığıdır. **provider** bloğu platforma bağlanmak için gereken ayarları içerir: bölge ve örneğin bütün kaynaklara otomatik eklenecek etiketler. Kimlik bilgileri bu bloğa ya da herhangi bir .tf dosyasına yazılmaz; ortam değişkenleri, roller ya da OIDC kullanılır. **resource** bloğu ise Terraform'un oluşturacağı ve yöneteceği nesneyi tanımlar; bir kaynak türü ve yapılandırma içinde ona referans vermek için kullanılan yerel bir ad taşır.

**data** bloğu yeni bir kaynak oluşturmaz; var olan bir bilgiyi okur. Örneğin bir bölgedeki en güncel işletim sistemi image'ı bir data source ile bulunur ve sanal makine kaynağında kullanılır; böylece bölgeye bağımlı, elle yazılmış ve zamanla eskiyen kimliklerden kaçınılır. Kısa kural şudur: resource yönetir, data okur.

Terraform kaynaklar arasındaki bağımlılıkları referanslardan çıkarır. Bir sanal makine bir data source'un sonucunu kullanıyorsa Terraform önce onu okuması, sonra makineyi oluşturması gerektiğini anlar; bağımsız kaynakları ise paralel oluşturur. Sıranın elle belirtilmesi çoğu zaman gerekmez.

## State

**State**, Terraform'un plan üretirken kullandığı bilinen durumdur: kaynak kimlikleri, özellikleri ve yapılandırmayla eşleşmeleri. Üç dünya vardır: ne olmasını istediğinizi söyleyen yapılandırma, Terraform'un bildiği eşleşmeleri tutan state ve provider'ın yönettiği gerçek kaynaklar. `terraform plan` yapılandırmayı state ile karşılaştırır ve gerekirse gerçek platformdan state'i tazeler. State olmasaydı Terraform yapılandırmadaki bir kaynağın bulutta hangi kaynağa karşılık geldiğini bilemezdi.

State varsayılan olarak çalışma dizininde bir dosyada tutulur; kişisel denemeler için bu yeterlidir. Ekip çalışmasında ise ortak bir **remote backend** kullanılır; AWS'de bu genellikle S3'tür. Bir de **locking** gerekir: iki kişi aynı anda değişiklik uygularsa state bozulabilir; kilitleme, aynı state üzerinde eşzamanlı değişikliği engeller. Remote backend'de erişim, şifreleme, kilitleme ve izinler birlikte tasarlanır.

State dosyası kaynak özelliklerini ve veritabanı parolaları gibi hassas değerleri içerebilir. Bu yüzden kod ile state farklı saklanır: .tf dosyaları, lock dosyası ve örnek değişken şablonları Git'e girer; state dosyaları ve gerçek secret içeren değişken dosyaları girmez. State'e erişim, kod deposuna erişimden daha dar tutulmalıdır.

State'i yönetmenin araçları da vardır. State'teki kaynaklar listelenebilir ve tek tek incelenebilir. **moved** bloğu, bir kaynağı yeniden adlandırırken ya da bir modüle taşırken Terraform'un onu silip yeniden oluşturmasını engeller. **import** bloğu, arayüzden elle oluşturulmuş bir kaynağı Terraform yönetimine alır. **removed** bloğu ise bir kaynağı silmeden state'ten çıkarır. Kural şudur: state dosyası elle düzenlenmez; değişiklikler kod üzerinden, bu bloklarla ve incelemeyle yapılır.

**Drift**, birisi arayüzden bir değişiklik yaptığında gerçek dünyanın kodla ayrışmasıdır; örneğin bir güvenlik grubuna tüm internetten SSH erişimi eklenmesi. Yalnızca tazeleme yapan bir plan bu değişiklikleri gösterir. İki seçenek vardır: normal bir plan kodu esas alır ve drift'i geri alacak değişikliği önerir; yalnızca tazeleme yapan bir apply ise gerçeği kabul edip state'i günceller, ardından kodun da düzeltilmesi gerekir. Bir güvenlik açığı oluşturan drift'te kodun kazanması gerektiği açıktır. Drift'i önlemek için arayüz yetkileri daraltılır ve zamanlanmış planlarla drift alarmı kurulur. Aynı kavram GitOps'ta da karşımıza çıkar.

## Yaşam döngüsü ve güvenli iş akışı

Terraform'un yaşam döngüsü beş adımdır: HCL yazmak, init, validate, plan ve apply. Bu bir komut ezberi değil, güvenli altyapı değişikliği alışkanlığıdır.

`terraform init` yeni bir yapılandırma dizininde çalıştırılan ilk komuttur; provider eklentilerini indirir ve backend'i hazırlar. `terraform fmt` kod biçimini standartlaştırır, `terraform validate` söz dizimini, referansları ve tipleri kontrol eder; ikisi de hiçbir kaynak oluşturmaz. `terraform plan` istenen durum ile bilinen durum arasındaki farkı gösterir: eklenecek, değişecek ve silinecek kaynakları ve en önemlisi silinip yeniden oluşturulacak, yani replace edilecek kaynakları. Plan yalnızca bir önizlemedir. Beklenmeyen bir silme ya da replace görülürse apply etmeden önce yapılandırma yeniden incelenmelidir; bir veritabanının replace edilmesi, verinin kaybolması demek olabilir. `terraform apply` onaylanan planı provider API'leri üzerinden uygular ve sonunda state'i günceller. Test ve geçici ortamlar için `terraform destroy` kaynakları siler; bulut maliyetleri açısından bu adım unutulmamalıdır.

Güvenli bir iş akışının kritik ilkesi, incelenen plan ile uygulanan planın aynı olmasıdır. Plan bir dosyaya kaydedilir, incelenir ve yalnızca o dosya uygulanır. Plan dosyası kullanılmadan önce plan sonra apply çalıştırılırsa, arada birisi altyapıyı ya da kodu değiştirmiş olabilir ve uygulanan değişiklik incelenenden farklı olur.

CI'da Terraform şöyle çalışır: bir merge request açıldığında biçim ve doğrulama kontrolleri, tflint ve altyapı kodu taraması çalışır, plan çıktısı merge request'e yorum olarak yazılır, birleştirme ve onaydan sonra kaydedilmiş plan uygulanır. CI buluta OIDC ile kısa ömürlü bir rol alarak erişir; plan rolü yalnızca okuma ve state erişimi, apply rolü yazma yetkisi taşır. Plan dosyası secret içerebileceği için saklama süresi kısa ve erişimi kısıtlı tutulur. Eşzamanlılık kilidi tek bir apply'ın çalışmasını sağlar; backend kilidi ikinci bir güvencedir.

## Tekrar kullanım: değişkenler, modüller

**Input değişkenleri**, aynı kodu farklı ortamlarda farklı değerlerle kullanmayı sağlar. Değişkenin tanımı ve tipi bir dosyada, ortama özel değerleri ise ayrı değişken dosyalarında tutulur. **Output**'lar Terraform'un oluşturduğu önemli bilgiyi terminale ya da başka araçlara sunar; örneğin oluşturulan bir sunucunun adresi Ansible'a ya da pipeline'ın sonraki adımına aktarılır. Secret değerler output olarak yazılırsa görünür hâle gelir ve hassas olarak işaretlenmelidir. **locals** ise yapılandırma içinde tekrar eden hesaplanmış değerleri, örneğin isim öneklerini ve ortak etiketleri, tek yerde tanımlar; değişken dışarıdan değer alır, local içeride hesaplanır.

Benzer kaynakları kopyalamadan üretmek için iki meta-argüman vardır. **count** basit sayısal tekrar içindir. **for_each** ise anlamlı anahtarlarla tanımlanmış bir küme içindir. count ile bir liste kullanıldığında listenin ortasından bir eleman silinirse sonraki bütün kaynakların sırası kayar ve Terraform onları gereksiz yere replace etmek ister; for_each anahtar kullandığı için bu sorun yaşanmaz.

**Modül**, tekrar kullanılabilir bir Terraform kod paketidir. Root modül ağ, hesaplama ve güvenlik gibi sorumlulukları alt modüllere ayırabilir. Bir modül çağrısı, modülün yerini belirten bir kaynak ve modülün input değişkenlerine verilen değerlerden oluşur; modülün output'ları ise çağıran tarafından kullanılır. Bir modülü iç yapısını bilmeden kullanabilmek için input ve output arayüzünün iyi tasarlanması gerekir. Modüller, platform ekibinin onaylı bir ağ ya da cluster modülü sunup ekiplerin onu kullanması yoluyla golden path fikrinin altyapıdaki uygulamasıdır.

Küçük örneklerde her şey tek bir dosyada olabilir, ama gerçek projelerde dosyalar sorumluluğa göre ayrılır: ana kaynaklar, değişkenler, output'lar, provider ve sürüm ayarları, backend ayarı, ortama özel değişken dosyaları ve modüller dizini. Terraform bir dizindeki bütün .tf dosyalarını birlikte okur; dosya adları okunabilirlik içindir.

## Terraform uygulamada

Bulut hesabı olmadan Terraform pratiği yapmanın en iyi yolu Docker provider'ıdır: aynı plan ve apply döngüsü, ama kaynaklar yerel Docker'da oluşturulur. Ağ, image ve container gibi kaynaklar kodla tanımlanır. Bu bir topluluk provider'ıdır ve sürümü mutlaka sabitlenmelidir. Docker Compose da benzer işleri yapabilir; fark yaşam döngüsündedir: Compose uygulama yığınını çalıştırmaya odaklanır, state tutmaz ve önizleme sunmaz; Terraform altyapıyla birlikte yönetir, state ile farkı hesaplar ve plan ile önizleme sunar. Yaygın yaklaşım uygulamalar için Compose ve Helm, altyapı için Terraform kullanmaktır.

Terraform ile Kubernetes ve Helm de yönetilebilir: namespace'ler, temel servisler ve Helm release'leri. Ama Terraform her şeyi yapabilir olsa da yapmamalıdır. Sınır şöyle çizilir: bulut ağı ve cluster'ın kendisi uzun ömürlü altyapıdır ve Terraform ile yönetilir. GitOps aracı ve ingress gibi cluster eklentileri Terraform ile bir kez kurulabilir, sonrası GitOps'a bırakılır. Sık değişen uygulamalar ise GitOps ve Helm ile dağıtılır; her image güncellemesi bir Terraform apply olmamalıdır. Secret değerleri ise state'e girmemesi için bir secret yöneticisiyle yönetilir. Kısaca: Terraform cluster'ı ve GitOps aracını kurar, uygulamaları GitOps dağıtır.

Hataları apply'dan önce yakalamak için bir kalite zinciri kurulur: biçim kontrolü, doğrulama, geçersiz instance tipleri ve kullanılmayan değişkenler gibi sorunları bulan tflint, açık depolama ve şifrelenmemiş diskler gibi güvenlik sorunlarını bulan Trivy, kuruma özel kurallar için OPA ve değişikliğin aylık maliyetini pull request'te gösteren Infracost gibi araçlar.

Terraform'da sık karşılaşılan hatalar da mesajlarından tanınır: başka bir apply çalıştığında ya da yarım kalmış bir kilit olduğunda state kilidi alınamaz; provider değiştiğinde lock dosyası güncel olmaz; bulut kimliği yoksa kimlik bilgisi bulunamaz; değiştirilemeyen bir alan ya da count kayması beklenmeyen replace'lere yol açar; elle oluşturulmuş bir kaynak "zaten var" hatası verir ve import ile çözülür. Ayrıntılı loglama açık bırakıldığında secret içerebilir. Kaçınılması gereken anti-pattern'ler ise state'i Git'e koymak, plan okumadan apply etmek, bütün ortamlar için tek bir dev state kullanmak, provider sürümünü serbest bırakmak, listelerle count kullanmak, arayüzden düzeltme yapmak, secret'ları .tf dosyalarına yazmak ve sunucu kurulumu için provisioner'lara dayanmaktır.

> IaC altyapı kararlarını görünür, incelenebilir ve tekrarlanabilir kılar. Terraform altyapının yaşam döngüsünü, Ansible sunucu yapılandırmasını yönetir. State kod ile gerçek dünya arasındaki köprüdür ve korunmalıdır; incelenen plan ile uygulanan plan aynı olmalıdır.
