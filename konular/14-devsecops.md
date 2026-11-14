# DevSecOps

DevSecOps, güvenliği teslimatın sonundaki bir manuel onaydan çıkarıp pipeline'ın doğal bir parçası hâline getirmektir. SRE, platform ve DevSecOps konusunda kavramına giriş yaptık; bu dokümanda tehdit modellemeyi, OWASP risklerini, statik analizi, bağımlılık ve image taramasını, secret yönetimini, yazılım tedarik zinciri güvenliğini ve politika olarak kodu araçlarıyla birlikte ele alıyoruz.

## Temeller

DevSecOps üç ekibin ortak sürecidir. Geliştirme güvenli kod yazar, bağımlılıkları yönetir ve bulguları pull request üzerinden düzeltir. Güvenlik riskleri erken tespit eder ve politika ile güvenlik kapılarını otomatikleştirir. Operasyon güvenli artifact'i çalıştırır, izler ve olaylardan gelen geri bildirimi sürece taşır. DevSecOps daha fazla güvenlik aracı demek değildir; güvenlik kararlarının teslimat akışına gömülmesidir.

Her yaşam döngüsü aşamasına uygun bir güvenlik kontrolü eklenir: kod aşamasında statik analiz ve secret taraması en hızlı geri bildirimi verir; build sırasında bağımlılık, altyapı kodu ve container riskleri doğrulanır; release'de artifact yazılım envanteri ve imza ile kimlik kazanır; dağıtım öncesinde admission politikası son güvenlik sınırını uygular; işletmede yeni riskler geliştirme sürecine geri taşınır.

Araçlardan önce dört prensip gelir. **Paylaşılan sorumluluk**: geliştirme, güvenlik ve platform ekipleri güvenlik çıktılarının ortak sahibidir. **Kanıtı otomatikleştirmek**: tarama sonuçları, yazılım envanteri, imza ve politika kararları pipeline artifact'i olarak saklanır. **Fail fast**: yüksek riskli bir bulgu mümkün olan en erken noktada teslimatı durdurur. **Sürekli geri bildirim**: yeni açıklar ve çalışma anı bulguları sürüm çıktıktan sonra da değerlendirilir. Bir güvenlik kapısı "her şeyi engelle" demek değildir; risk eşiğini tutarlı biçimde uygulamaktır. Her şeyi engelleyen bir kapı kısa sürede devre dışı bırakılır.

**Tehdit modelleme**, pipeline'a araç eklemeden önce neyin korunduğunu ve saldırı yüzeyini anlamaktır. Dört soru sorulur: hangi varlıkları koruyoruz (kaynak kod, CI token'ları, imza kimliği, registry, cluster kimlik bilgileri, müşteri verisi); hangi tehditler var (secret sızıntısı, ele geçirilmiş bağımlılık, değiştirilmiş image, yetki yükseltme, istenmeden açığa çıkma); hangi kontroller bu tehditleri azaltıyor (çok faktörlü doğrulama, kod incelemesi, tarama, imza, politika, en az yetki, izleme) ve kontrolün çalıştığını hangi kanıtlar gösteriyor (tarama raporu, yazılım envanteri, imza, politika kararı, denetim kaydı). Belirli bir tehdidi azaltmayan bir kontrol, yalnızca pipeline maliyetidir.

Erken bulmanın ekonomisi güvenlikte de geçerlidir: aynı açık bir kod incelemesinde bulunursa dakikalar, üretimde bulunursa olay müdahalesi, yama, yeniden dağıtım, itibar ve uyum maliyeti demektir. Shift-left bulguyu yazıldığı yerde yakalar; **shift-right** ise üretimden gelen sinyali koda geri döndürür.

Tek bir araç her şeyi görmez; güvenlik katmanlı bir savunma gerektirir:

| Tür | Ne inceler? | Ne zaman? | Örnek araç |
|---|---|---|---|
| Secret scanning | Kod ve geçmişteki parola ve token | Commit öncesi, PR | Gitleaks |
| SAST | Kaynak kod kalıpları | PR | SonarQube, Semgrep |
| SCA | Bağımlılık açıkları ve lisanslar | PR, build | Trivy, Dependabot |
| IaC scanning | Terraform, Kubernetes, Dockerfile ayarları | PR | Trivy, Checkov |
| Container scanning | Image'daki paketler | Build, registry | Trivy, Harbor |
| DAST | Çalışan uygulamaya saldırı | Staging | OWASP ZAP |
| Admission ve runtime | Cluster'a giriş ve çalışma anı davranışı | Dağıtım, üretim | Kyverno, Falco |

## OWASP Top 10

OWASP Top 10, web uygulamalarındaki en kritik güvenlik risklerinin listesidir ve 2025 sürümü veri ve topluluk anketiyle belirlenmiştir. Birinci sırada yetkisiz veri ve işlem erişimi, yani **broken access control** bulunur. İkinci sırada varsayılan ayarlar ve açık bırakılmış kaynaklar gibi **güvenlik yanlış yapılandırması** gelir. Üçüncü sırada yeni bir kategori olan **yazılım tedarik zinciri hataları** yer alır: bağımlılıklar, build sistemi ve dağıtım altyapısı. Listenin devamında şifreleme hataları, injection, güvensiz tasarım, kimlik doğrulama hataları, yazılım ve veri bütünlüğü hataları, loglama ve alarm eksikliği ve yine yeni bir kategori olan hata durumlarının güvensiz işlenmesi bulunur. Tedarik zincirinin üçüncü sıraya yükselmesi, son yıllardaki paket ve CI saldırılarının sonucudur.

Her kategorinin DevOps'ta bir karşılığı vardır. Erişim kontrolü yetki testleri, en az yetki ve Kubernetes RBAC ile; yanlış yapılandırma altyapı kodu ve manifest taramasıyla; tedarik zinciri bağımlılık taraması, yazılım envanteri, imza ve sabitlenmiş CI eylemleriyle; injection statik analiz kurallarıyla; kimlik doğrulama çok faktörlü doğrulama, tek oturum açma ve OIDC ile; bütünlük imzalı artifact'ler ve admission doğrulamasıyla; loglama eksikliği merkezi log ve güvenlik alarmlarıyla ele alınır.

## Secret yönetimi

En sık ve en pahalı hatalardan biri Git'e düşen bir anahtardır. Savunma beş adımdır: commit öncesi taramayla önlemek, pull request taramasıyla yakalamak, secret'ları bir kasada saklamak, kısa ömürlü kimlik kullanmak ve sızıntıda anahtarı döndürmek. Depoda secret bulunmaz, yalnızca örnek bir yapılandırma dosyası bulunur. Secret'lar Vault, bulut secret yöneticileri ya da CI secret depolarında, Kubernetes'te ise External Secrets ile tutulur. OIDC ve dinamik kimlikler, bir kimlik sızsa bile kısa sürede geçersiz olmasını sağlar.

**Gitleaks**, kodda ve Git geçmişinde secret arayan hızlı bir tarayıcıdır; bütün geçmişi, yalnızca çalışma dizinini ya da yalnızca bir pull request'teki commit'leri tarayabilir. Commit öncesi kanca olarak kurulduğunda secret geliştiricinin makinesinde, commit'lenmeden yakalanır. Üç katman birlikte çalışır: yerelde commit öncesi tarama, CI'da pull request taraması ve platformda, secret içeren gönderimleri anında reddeden push protection.

Bir secret sızdığında ilk kural şudur: silmek çözüm değildir, geçersiz kılmak çözümdür. Herkese açık depoları tarayan botlar sızan anahtarları dakikalar içinde bulabilir. Önce anahtar sağlayıcıda iptal edilir ve yenisi kasaya konur. Sonra kapsam belirlenir: anahtar ne kadar süre açıktaydı, depo herkese açık mıydı? Denetim kayıtlarında bu anahtarla yapılan çağrılar incelenir. Ardından geçmiş temizlenir ve bütün kopyaların yeniden alınması sağlanır. En son kök neden bulunup önlenir: commit öncesi tarama, push protection ve uzun ömürlü anahtarların OIDC ile tamamen kaldırılması. Olay suçlamasız bir postmortem ile kayda geçirilir.

## Statik analiz ve SonarQube

**SAST**, kod çalışmadan önce kaynak kod üzerinde güvenlik kuralları uygular. Injection kalıplarını, güvensiz API kullanımını ve kod seviyesindeki güvenlik sorunlarını bulur. Editörde, pull request'te ve CI'da çalışabilir; en hızlı geri bildirim pull request aşamasında alınır. Sınırı, çalışma anı davranışını, gerçek ağ akışını ve iş mantığı risklerinin tamamını görememesidir. SAST sonucu bir hata listesi değil, bir teslimat kararı üreten sinyaldir.

**SonarQube** güvenlik, güvenilirlik ve bakım kolaylığını tek ekranda gösterir. Doğrudan istismar edilebilir açıklar **vulnerability**, insan incelemesi gerektiren güvenlik açısından hassas kodlar **security hotspot**, yanlış davranışa yol açacak kodlar **bug**, bakımı zorlaştıran kodlar **code smell** olarak sınıflanır; ayrıca test kapsamı ve kod tekrarı ölçülür. Her boyut harf notları alır ve kalite kapısı bunlardan koşul üretir.

SonarQube'da üç kavram önemlidir. **Quality profile** hangi kuralların uygulanacağını belirler. **New code**, yeni ya da değiştirilmiş kodu ayrı izler. **Quality gate** ise koşullar sağlanmadığında pull request'i ya da pipeline'ı başarısız sayar. New code kavramı özellikle değerlidir: yıllanmış bir projeye analiz eklendiğinde binlerce bulgu çıkar ve ekip motivasyonunu kaybeder. Pratik yaklaşım, önce yeni kodun temiz kalmasını garanti etmek, sonra eski teknik borcu kademeli azaltmaktır. Kalite kapısı ekip olgunluğuna göre tasarlanır: güvenlik koşulları, örneğin yeni kodda sıfır vulnerability ve bütün hotspot'ların incelenmesi, baştan sıfır tolerans ile uygulanır; test kapsamı ve kod tekrarı gibi kalite koşulları ise kademeli sıkılaştırılır.

SonarQube üç CI aracıyla da entegre çalışır ve kalite kapısının sonucunu bekleyerek pipeline'ı durdurabilir. Yeni kodun doğru hesaplanması için pipeline'da tam Git geçmişinin çekilmesi gerekir. Gürültülü bir araç sonunda kapatıldığı için kural setinin yönetilmesi de önemlidir: hatalı pozitifler gerekçesiyle işaretlenir, bilinçli kabul edilen riskler süreli ve onaylı olur, üretilen kod ve üçüncü taraf kod analizden çıkarılır.

## Bağımlılık ve image taraması

**SCA** (Software Composition Analysis), lockfile'daki ya da image'daki her paketin bilinen açıklarını ve lisansını kontrol eder. Paket envanteri çıkarılır, açık veritabanlarıyla eşleştirilir ve şiddet ile düzeltme sürümü bulunur. Açıkların çoğu dolaylı bağımlılıklardadır. Lisans boyutu da önemlidir: bazı lisanslar ticari dağıtımda yükümlülük getirir.

Her açık aynı aksiyonu gerektirmez. Önceliklendirmede dört soru sorulur: açığın şiddeti ne, açıklı kod gerçekten uygulamanın yolunda çağrılıyor mu, bir düzeltme sürümü var mı ve bileşen internete açık mı, kritik veriye yakın mı? CVSS puanı bir açığın "ne kadar kötü olabileceğini" söyler; gerçekte istismar edildiği bilinen açıkların listesi (CISA KEV) ve önümüzdeki dönemde istismar edilme olasılığını tahmin eden puanlar (EPSS) ise "ne kadar olası" olduğunu söyler. Örnek bir kural: istismar edildiği bilinen ya da kritik olup düzeltmesi bulunan açıklar engellenir, diğerleri süreli olarak takip edilir.

**Trivy**, CI'da en yaygın kullanılan açık kaynak tarayıcılardan biridir ve birçok hedefi tarayabilir: depo ve dosya sistemi, container image, Terraform, Kubernetes, Dockerfile ve Helm yapılandırmaları, uzak Git depoları, çalışan bir cluster ve yazılım envanteri dosyaları. Image taraması bir CI kapısı olarak kullanılır: yüksek ve kritik açıklar için bir eşik aşılırsa pipeline durur, düzeltmesi henüz olmayan açıklar engellenmeden takip edilir, raporlar artifact olarak saklanır. Base image seçimi açık sayısını doğrudan etkiler. Harbor'da image gönderildikten sonra yeniden taranır, çünkü yeni açıklar her gün yayımlanır.

**IaC taraması**, yanlış güvenlik yapılandırmasını bulut kaynağı oluşmadan yakalar: Terraform'da herkese açık kaynaklar, tüm internete açık güvenlik grupları ve şifrelenmemiş diskler; Kubernetes'te ayrıcalıklı container'lar, root kullanıcılar ve eksik güvenlik ayarları; Dockerfile'da root kullanıcı ve latest etiketi.

Her bulgu düzeltilemeyebilir; istisna gerekir. Ama istisnaların kuralları vardır: gerekçeli, süreli ve incelenmiş olmalıdır. Bir istisna dosyasında açığın kimliği, gerekçesi, ilgili iş kaydı ve bitiş tarihi bulunur; tarih geçince bulgu geri gelir. **VEX** ise makinece okunabilir bir "bu üründe etkilenmiyoruz" beyanıdır. İstisna dosyaları güvenlik ekibinin onayından geçmelidir; aksi hâlde herkes kendi bulgusunu susturur.

## Yazılım tedarik zinciri

Tedarik zinciri güvenliği üç soruya cevap verir: ne üretildi, kim üretti ve nasıl üretildi?

**SBOM** (Software Bill of Materials), bir yazılımın içindekiler listesidir; bir gıda paketinin arkasındaki içerik listesine benzer. Syft ya da Trivy gibi araçlarla üretilir ve CycloneDX ile SPDX olmak üzere iki yaygın standart formatı vardır. Her build'de image ile birlikte üretilir ve saklanır. Değeri, yeni bir açık yayımlandığında ortaya çıkar: "bu kütüphane hangi image'larımızda var?" sorusu dakikalar içinde cevaplanır. Regülasyonlar da yazılım envanterini giderek zorunlu hâle getirmektedir.

**Cosign** ile image'ın digest'i imzalanır. Anahtarsız modda kalıcı bir özel anahtar bulunmaz: CI'ın OIDC kimliği kısa ömürlü bir sertifikaya yazılır ve imza bir şeffaflık kaydına eklenir. Yazılım envanteri ve build bilgisi de imzalı ekler (attestation) olarak image'a bağlanabilir. Doğrulama yapılırken imzalayan kimlik ve sağlayıcı mutlaka kontrol edilmelidir; "herhangi bir imza var mı?" sorusu yeterli değildir. İnternetsiz ortamlarda anahtarlı imza kullanılır.

OWASP'ın CI/CD güvenliği için ayrı bir risk listesi de vardır, çünkü saldırgan için CI/CD üretime giden en kısa yoldur. Riskler arasında incelemesiz dağıtım gibi yetersiz akış kontrolü, zayıf kimlik ve erişim yönetimi, bağımlılık zinciri istismarı, bir pull request ile pipeline tanımının değiştirilip secret'ların çalındığı zehirli pipeline yürütme, aşırı pipeline yetkileri, zayıf credential hijyeni, güncellenmemiş runner'lar, denetimsiz üçüncü taraf servisler, doğrulanmayan artifact bütünlüğü ve yetersiz loglama bulunur.

Pipeline'ı sertleştirmenin ilkeleri üç CI aracında da aynıdır: en az yetki, statik anahtarlar yerine OIDC ile kısa ömürlü kimlik, hazır eylemlerin commit kimliğiyle sabitlenmesi, geçici ve izole runner'lar, ayrıcalıklı Docker yerine rootless build araçları, fork'tan gelen pull request'lerin secret içeren job'lara erişememesi ve üretim ortamında onaylayıcı ve branch kısıtı. Pipeline tanımı da koddur ve değişiklikleri platform ekibinin onayından geçmelidir.

## Uçtan uca güvenli teslimat

Şimdiye kadar anlatılanlar tek bir akışta birleşir.

Pull request aşamasında secret taraması ve branch protection, statik analiz aşamasında SonarQube ve kalite kapısı, build aşamasında temiz kurulum, bağımlılık taraması ve değişmez bir digest, image kapısında kritik açık filtresi, Harbor'a gönderim ve değiştirilemez etiket, son olarak staging testi, üretim onayı ve kademeli dağıtım. Temel kural, ortamlar arasında yeni image derlenmemesidir.

Her bulgu build'i durdurmaz. Secret bulguları, başarısız testler, yeni koddaki güvenlik açıkları, düzeltmesi bulunan kritik bağımlılık açıkları ve image'daki yüksek ve kritik açıklar engelleyicidir. Küçük kod kalitesi sorunları yalnızca bilgilendirir. Yazılım envanteri ise görünürlük için artifact olarak saklanır. Güvenlik kapıları sürüm kontrolünde tutulur ve istisnalar süreli, onaylı ve gerekçeli olur.

## Politika olarak kod

Pipeline kapıları erken geri bildirim verir, ama birisi pipeline'ı atlayıp doğrudan cluster'a bir değişiklik uygularsa ne olur? Bunun cevabı **admission kontrolüdür**.

Bir istek API server'a gelir, kimlik ve yetki kontrol edilir, mutating admission isteği değiştirebilir, validating admission isteği kabul ya da reddeder ve en son durum etcd'ye yazılır. Pipeline kapısı yalnızca pipeline'dan geçen değişiklikleri görür; admission kapısı ise her yoldan gelen isteği denetler. Politikalar önce raporlama modunda çalıştırılır, ihlaller temizlenir, sonra zorunlu hâle getirilir. En iyi yaklaşım ikisini birlikte kullanmaktır.

**Kyverno**, Kubernetes için bir politika motorudur ve CNCF projesidir. Yeni bir dil öğrenmeden, Kubernetes kaynakları gibi YAML ile politika yazmayı sağlar. Dört yeteneği vardır: kuralı ihlal eden isteği reddetmek ya da raporlamak (validate), eksik alanları eklemek (mutate), yeni bir namespace açıldığında ağ politikası ve kota gibi kaynakları otomatik oluşturmak (generate) ve image imzalarını doğrulamak (verifyImages). Son yetenek tedarik zincirinin son halkasıdır: belirli bir registry'den gelen image'ların kurumun kimliğiyle imzalanmış olması şart koşulur, etiketler digest'e çevrilir ve pipeline dışından gönderilmiş imzasız bir image cluster'da çalışamaz.

Bir denetimde "pipeline yeşildi" demek yeterli değildir; kanıt gösterilmelidir. Kanıt zinciri commit kimliğinden pipeline kimliğine, image digest'inden yazılım envanterine ve tarama sonucuna, imzadan dağıtım kaydına uzanır. Bu zincir, üretimdeki her Pod'un hangi commit'ten geldiğini, kimin onayladığını ve hangi taramadan geçtiğini gösterir; denetimlerde, olay müdahalesinde ve müşteri talep ettiğinde kullanılır.

DevSecOps da ölçülür. Kritik ve yüksek açıkların kapanma süresi, hizmet seviyesini aşmış açık bulgu sayısı, güvenlik kapısında kırılan pipeline oranı, aktif ve süresi geçmiş istisna sayısı, üretimdeki imzalı ve envanterli image oranı ve sızan secret sayısı izlenir. Güvenlik kapısında kırılma oranının önce artıp sonra düşmesi normaldir: kapı konduğunda gizli bulgular ortaya çıkar, sonra ekip öğrenir.

> DevSecOps güvenliği erken başlatır, artifact'e güven inşa eder ve politikayla otomatik korur. Tarama, yazılım envanteri, imza ve admission politikası birlikte, üretimde neyin çalıştığını ve nereden geldiğini kanıtlanabilir kılar. DevSecOps güvenlik taraması eklemek değil, güvenilir bir teslimat sistemi tasarlamaktır.
