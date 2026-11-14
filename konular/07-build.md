# Build, bağımlılık yönetimi ve test

Kaynak kod tek başına çalışmaz. Bağımlılıklarının indirilmesi, derlenmesi, test edilmesi ve paketlenmesi gerekir. Bu adımların toplamına **build** denir ve CI/CD pipeline'larının çekirdeğinde hep bu adımlar çalışır. Bu dokümanda build araçlarının ortak kavramlarını, bağımlılık yönetimini, örnek olarak npm ekosistemini ve test otomasyonunu ele alıyoruz.

## Build nedir?

Build, kaynak kod ile çalıştırılabilir paket arasındaki her adımdır: kütüphaneleri indirip sürüm çakışmalarını çözmek, Java'yı bytecode'a, Go'yu ikili dosyaya ya da TypeScript'i JavaScript'e derlemek, ön yüz dosyalarını birleştirip küçültmek, testleri çalıştırmak, bir jar, ikili dosya ya da container image'ı olarak paketlemek ve bir registry'ye yayınlamak. Python ya da JavaScript gibi yorumlanan dillerde derleme adımı olmayabilir, ama bağımlılık çözümü, test ve paketleme yine build sürecinin parçasıdır.

Build aracı bu adımları tek bir komutla ve her makinede aynı şekilde çalıştırır. Bu yüzden bir geliştiricinin editöründeki "Build" düğmesi yeterli değildir: editör ayarları kişiye özeldir, pipeline ise yalnızca komut satırını bilir. İyi bir build tek komutla çalışır, tekrarlanabilirdir ve komut satırından yürütülür.

## Manifest, lockfile ve registry

Farklı dillerin build araçları farklı isimler taşır: JavaScript'te npm, Java'da Maven ve Gradle, Python'da pip ve uv, Go'da go modülleri. Ama her ekosistemde aynı üç kavram vardır. **Manifest**, hangi bağımlılıkların istendiğini yazdığınız dosyadır: package.json, pom.xml, go.mod. **Lockfile**, tam olarak hangi sürümlerin kurulduğunu kaydeden dosyadır: package-lock.json, go.sum. **Registry** ise paketlerin indirildiği yerdir: npm registry, Maven Central, PyPI. Yeni bir dil öğrenirken ilk soru şu olmalıdır: manifest ne, lockfile ne ve paketler nereden geliyor?

Bir projeye birkaç bağımlılık eklendiğinde aslında yüzlerce paket gelir, çünkü her bağımlılığın kendi bağımlılıkları vardır.

Manifest'e yazılanlara **doğrudan**, onların bağımlılıklarına **dolaylı** (transitive) bağımlılık denir. Güvenlik açısından bu ayrım önemlidir: bir güvenlik açığı çoğu zaman eklediğiniz pakette değil, onun bağımlılıklarındadır.

Paketlerin sürümleri **semantic versioning** ile numaralandırılır: üç sayıdan ilki geriye uyumsuz değişikliği (major), ikincisi geriye uyumlu yeni özelliği (minor), üçüncüsü geriye uyumlu hata düzeltmesini (patch) gösterir. Manifest'te sürüm aralıkları yazılabilir: tam sabit bir sürüm, yalnızca patch güncellemelerini kabul eden tilde işareti ya da minor ve patch güncellemelerini kabul eden şapka işareti. Örneğin `^1.4.2`, 1.4.2 ve sonrasındaki ama 2.0.0'ın altındaki bütün sürümleri kabul eder. 1.0 öncesindeki sürümlerde her minor güncelleme kırıcı sayılır.

Aralık neyin kabul edildiğini, lockfile ise şu an tam olarak neyin kurulu olduğunu söyler. Lockfile'da her paketin tam sürümü, indirildiği adres ve içeriğinin bir özeti (integrity hash) bulunur; birisi registry'deki paketi değiştirmişse özet tutmaz ve kurulum durur. Lockfile depoya commit edilir; onu .gitignore'a koymak tekrarlanabilirliği ortadan kaldırır. Lockfile olmayan bir projede bir gün mutlaka "dün çalışıyordu" cümlesi kurulur, çünkü dolaylı bir bağımlılık gece yeni bir sürüm yayınlar ve sabah build kırılır.

Kurumsal ortamlarda pipeline'lar paketleri doğrudan internetten değil, bir iç proxy depodan çeker: Nexus, Artifactory ya da container imajları için Harbor. Bu depo dış registry'leri önbelleğe alır, böylece dış servis çökse bile build çalışır; hangi paketin kullanıldığı kayıt altına alınır ve zararlı paketler engellenebilir; şirketin kendi kütüphaneleri de dışarı çıkmadan paylaşılır.

## Tedarik zinciri riskleri

Bir bağımlılık eklemek, başkasının kodunu kendi makinenizde ve pipeline'ınızda çalıştırmak demektir. Dört temel risk vardır. **Typosquatting**, popüler bir pakete benzer isimli sahte bir paket yayınlamaktır. **Ele geçirilmiş hesap**, popüler bir paketin sahibinin hesabı ele geçirilip zararlı bir sürüm yayınlanmasıdır; geniş bir sürüm aralığı bu sürümü otomatik olarak çeker. **Kurulum script'leri**, paket kurulurken kod çalıştırır ve token ya da secret çalmak için kullanılabilir. **Terk edilmiş paketlerde** ise güvenlik açıkları kapanmaz.

2025'te yaşanan ve kendi kendini çoğaltan bir npm solucanı bu risklerin birleşimini gösterdi: ele geçirilen paket sahibi hesaplarıyla popüler paketlere kurulum script'leri eklendi, kurulduğu makinelerde token'lar çalındı ve solucan yeni paketlere bulaştı. Savunma katmanlıdır: lockfile, temiz ve kilitli kurulum, iç proxy depo, bağımlılık taraması ve en az yetkili CI token'ları.

## npm ekosistemi

Node.js ekosisteminde **node** JavaScript çalışma ortamı, **npm** onunla birlikte gelen paket yöneticisi, **npx** bir paketi kurmadan çalıştıran araç, **package.json** projenin manifest'i ve **node_modules** kurulu paketlerin klasörüdür; bu klasör depoya girmez. Pipeline'da kullanılan Node sürümü sabitlenmelidir; "hangi Node sürümüyle build ettik?" sorusunun cevabı belli olmalıdır.

package.json projenin kimliğini, komutlarını ve bağımlılık sözleşmesini taşır. `name` ve `version` paketin kimliğidir. `private: true` yanlışlıkla genel registry'ye yayınlanmasını engeller. `engines` desteklenen Node sürümünü belirtir. `scripts` bölümü pipeline'ın çağıracağı ortak komutları tanımlar: başlatma, lint, test ve build. `dependencies` çalışma zamanında gereken paketleri, `devDependencies` ise yalnızca geliştirme ve testte gereken paketleri listeler. Bu ayrım önemlidir: test araçlarını üretim imajına koymak imajı büyütür ve saldırı yüzeyini genişletir. Çok aşamalı container build'lerinde ilk aşamada her şey kurulup test edilir, son aşamada yalnızca üretim bağımlılıkları kurulur.

npm scripts'in değeri, geliştiricinin ve pipeline'ın aynı komutu çalıştırmasıdır. Pipeline tanımında uzun shell komutları yazmak yerine `npm run lint` ya da `npm test` çağrılır; böylece mantık depoda ve araçtan bağımsız kalır.

Bu bölümün en pratik ayrımı `npm install` ile `npm ci` arasındadır. `npm install` geliştirme içindir: paket ekler ya da günceller, gerekirse lockfile'ı değiştirir ve mevcut node_modules klasörünü korur. `npm ci` ise CI ve build içindir: lockfile'ı asla değiştirmez, package.json ile uyuşmuyorsa hata verir, önce node_modules'ü tamamen siler ve lockfile'daki tam bağımlılık ağacını kurar. Pipeline'da `npm install` görmek, lockfile kaymasına kapı açıldığının işaretidir. `npm ci` hata verdiğinde bu bir arıza değil, geliştiricinin lockfile'ı güncelleyip commit etmeyi unuttuğunun uyarısıdır.

Pipeline'ı hızlandırmanın en kolay yolu önbellektir, ama neyin önbelleğe alınacağını bilmek gerekir: node_modules değil, npm'in indirme önbelleği. Önbellek anahtarı lockfile'ın özeti olur; lockfile değişince önbellek yenilenir. Önbellek hızlandırır ama doğruluk kaynağı değildir; silindiğinde build yine aynı sonucu vermelidir.

npm'in güvenlik araçları da vardır: bilinen açıkları raporlayan ve belirli bir seviyede pipeline'ı kıran `npm audit`, registry imzalarını doğrulayan `npm audit signatures` ve kurulum script'lerini çalıştırmayan `--ignore-scripts` seçeneği. Bağımlılık güncellemeleri Dependabot ya da Renovate gibi araçlarla pull request olarak gelebilir ve pipeline tarafından test edilir. npm tek seçenek değildir; pnpm, Yarn ve Bun aynı package.json'u farklı stratejilerle kurar. Önemli olan, ekibin tek bir paket yöneticisinde anlaşmasıdır; iki lockfile iki farklı bağımlılık ağacı demektir.

## Test otomasyonu

Testler bir piramit olarak düşünülür. En altta kod çalışmadan önce hataları yakalayan **statik analiz** vardır: lint, tip kontrolü ve biçim kontrolü. Üstünde tek bir fonksiyonu izole olarak sınayan, milisaniyeler içinde çalışan **birim testleri**, onun üstünde modüllerin, veritabanının ve API'lerin birlikte çalıştığını doğrulayan **entegrasyon testleri**, en tepede gerçek bir tarayıcıda kullanıcı akışını sınayan **uçtan uca (E2E) testler** bulunur.

Aşağıda hız, yukarıda güven vardır: binlerce birim testi bir dakikada biter, uçtan uca testler ise gerçek kullanıcıya en yakın senaryoyu sınar. Testlerin çoğunun uçtan uca olduğu ters piramit, pipeline'ı yavaş ve kırılgan yapar; buna "dondurma külahı" denir. Genel kural, her hatayı yakalayabilecek en alt seviyede test yazmaktır.

**Statik analiz** en ucuz testtir. ESLint gibi araçlar kullanılmayan değişkenler ve unutulmuş await ifadeleri gibi hata kalıplarını, Prettier gibi araçlar biçim tutarsızlıklarını, TypeScript gibi diller tip hatalarını kod çalışmadan bulur. CI'da uyarıların birikmesine izin verilmemeli, aynı kontroller commit öncesinde de çalıştırılabilmelidir.

**Birim testi** tek bir davranışı dış dünyadan izole ve hızlı bir şekilde doğrular. İyi bir birim testi üç adımdan oluşur: hazırla, çalıştır, doğrula. Ağ, saat ya da rastgelelik içermez; aynı girdi her zaman aynı sonucu verir. Test isimleri davranışı anlatır, böylece kırılan bir testin ismi neyin bozulduğunu söyler. Birim testleri, kodu yeniden düzenlerken davranışın bozulmadığını garanti eden bir güvenlik ağıdır.

**Coverage**, testlerin kodun ne kadarını çalıştırdığını ölçer: satır, dal (if-else'in iki yolu), fonksiyon ve ifade kapsamı. Dal kapsamı genellikle en düşük ve en anlamlı olanıdır. Pipeline'da bir eşik koymak mümkündür. Ama coverage testlerin kalitesini değil, yalnızca kodun çalıştırılıp çalıştırılmadığını ölçer; hiçbir doğrulama içermeyen bir test de coverage'ı artırır. Düşük coverage bir uyarıdır, yüksek coverage bir garanti değildir.

Test sonuçlarının pipeline'da okunabilir olması gerekir. Sonuçlar JUnit XML gibi ortak bir formatta üretilirse Jenkins, GitLab ve GitHub Actions bunları test sekmesinde gösterir. İyi bir hata mesajı, kodu açmadan sorunu anlatır: hangi test, hangi beklenti, gelen değer ne? Kırık bir ana dal ise her zaman önceliklidir; ana dal kırmızıyken yeni iş eklenmez, ya düzeltilir ya da geri alınır.

**Continuous testing**, testin tek bir aşama değil, pipeline boyunca dağılmış kapılar olduğunu anlatır. Commit öncesinde lint ve ilgili birim testleri saniyeler içinde, pull request'te lint, tip kontrolü, birim ve entegrasyon testleri ile statik güvenlik analizi dakikalar içinde çalışır ve birleştirmeyi engelleyebilir. Staging dağıtımından sonra smoke ve kritik uçtan uca testler çalışır. Gece tam uçtan uca ve performans testleri koşar ve rapor üretir. Canlıda ise sentetik izleme sürekli çalışır. Pull request pipeline'ının on dakikayı geçmemesi önemlidir; geçerse geliştiriciler beklemek yerine başka işe geçer ve görev değiştirme israfı başlar.

## Pipeline'ı tasarlamak

Bir commit'ten üretime giden tam bir hat şu adımlardan oluşur: temiz kurulum, lint ve tip kontrolü, birim testleri, build, container image üretimi, image taraması, registry'ye gönderim, staging'e dağıtım, smoke testleri ve onaylı üretim dağıtımı.

İyi bir tasarım ilkesi, pipeline tanımının ince, mantığın ise depodaki bir script'te olmasıdır. Kurulum, lint, test, build ve image üretimini yapan bir script yazılır ve Jenkins, GitLab CI ya da GitHub Actions bu script'i tek satırla çağırır. Bu yaklaşım araç değiştirmeyi kolaylaştırır ve "pipeline'da kırıldı ama yerelde çalışıyor" tartışmasını bitirir, çünkü aynı script yerelde de çalışır. Araçlara özgü özellikler, yani önbellek, artifact ve ortam tanımları, pipeline tanımında kalır.

Pipeline'larda tekrar eden hatalar da vardır: pipeline'da `npm install` kullanmak, her ortam için yeniden build etmek ve böylece test edilen paketle dağıtılan paketin farklılaşmasına yol açmak, hangi sürümün çalıştığını belirsizleştiren `latest` etiketini kullanmak, sabit süre bekleyen ve bu yüzden kararsız sonuç veren testler yazmak, kırmızı bir ana dala alışmak, token'ları depoda tutmak, her şeyi tek bir job'a koymak ve yalnızca coverage için test yazmak.

> Build her makinede aynı sonucu veren tek bir komut olmalıdır. Lockfile ve temiz kurulum tekrarlanabilirliği, test piramidi hızlı ve güvenilir geri bildirimi sağlar. İyi bir pipeline hızlı, güvenilir ve sıkıcıdır; kimse ondan korkmaz.
