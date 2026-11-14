# GitHub Actions ve GitLab CI

Jenkins ayrı bir sunucudur. GitHub Actions ve GitLab CI ise kodun durduğu platformun içindedir: depo, pull ya da merge request ve pipeline aynı yerde yaşar. Bu dokümanda iki aracın kavramlarını, birbirine nasıl eşlendiğini ve güvenli kullanımını ele alıyoruz.

## GitHub Actions'ın temelleri

GitHub Actions, depodaki olayları otomasyona bağlayan bir workflow ve runner platformudur. Workflow dosyaları kodla aynı depoda, `.github/workflows` dizininde YAML olarak durur. Push, pull request, sürüm yayınlama, zamanlama ya da elle tetikleme gibi bir **event** bir **workflow**'u başlatır. Workflow **job**'lardan, job'lar da sırayla çalışan **step**'lerden oluşur. Bir step ya hazır bir eylemi (`uses`) ya da bir komutu (`run`) çalıştırır. Job'ları çalıştıran makineye **runner** denir; Jenkins'teki agent'ın karşılığıdır.

Bir workflow dosyasında `name` adı, `on` tetikleyiciyi, `permissions` otomatik oluşturulan token'ın yetkilerini, `jobs` ise yürütme grafiğini tanımlar. Workflow değişikliği de normal kod gibi incelemeden ve branch protection'dan geçmelidir, çünkü workflow dosyasını değiştirebilen biri secret'lara erişebilir.

Sık kullanılan event'ler şunlardır: branch ya da tag değişikliğini yakalayan push, değişiklik önerisini doğrulayan pull_request, Actions sekmesinde bir düğme oluşturan workflow_dispatch, zamanlanmış çalışma için schedule, sürüm yayınlandığında çalışan release ve başka bir workflow tarafından çağrılan yeniden kullanılabilir workflow'lar için workflow_call. Pull request doğrulaması ile üretim dağıtımı aynı event ve yetki modelini kullanmamalıdır; pull request'i herkes açabilir, üretime dağıtım yetkisi ise sınırlı olmalıdır.

## Secret'lar, değişkenler ve kimlik

GitHub Actions'ta değerler farklı yerlerde durur. **Variable**'lar gizli olmayan ayarlardır ve loglarda görünür. **Secret**'lar şifrelenir ve loglarda maskelenir. **GITHUB_TOKEN** her workflow çalışması için otomatik oluşturulur ve çalışma bitince geçersiz olur. **GitHub App token**'ları seçili depolar ve izinler için üretilir ve kişisel erişim token'larına tercih edilir; kişisel token'lar bir kişiye bağlıdır, kişi ayrılınca bozulur ve genellikle gereğinden fazla yetkilidir. **OIDC token**'ları ise bulut sağlayıcılarına anahtarsız giriş için kullanılır.

Fork'tan gelen pull request'lere secret verilmez ve otomatik token yalnızca okuma yetkisiyle gelir; aksi hâlde biri bir pull request açıp secret'ları çalabilirdi. **Environment secret**'ları ise yalnızca o ortama dağıtım yapan job tarafından ve yalnızca onaydan sonra görülebilir.

Workflow'larda kullanılan her hazır eylem, pipeline'a başkasının kodunu ekler. Bu yüzden eylemler sürümleriyle değil, tam commit kimlikleriyle sabitlenmelidir; branch'ler ve tag'ler sonradan başka bir commit'e taşınabilir, commit kimliği değişmez. 2025'te popüler bir eylemin tag'leri zararlı bir commit'e taşındığında birçok depoda secret'lar loglara yazdırıldı; commit kimliğiyle sabitleyen depolar etkilenmedi. Sabitlenmiş eylemlerin güncellemeleri Dependabot ile pull request olarak alınabilir. Organizasyon düzeyinde yalnızca izin verilen eylemlere izin vermek ve workflow izinlerini varsayılan olarak okumaya çekmek diğer önlemlerdir.

## Job'lar, runner'lar ve yürütme

Job ile step arasındaki fark çok önemlidir. Bir **job** kendi runner'ında, yani ayrı bir makinede çalışır; bağımlılıkları ve çıktıları vardır. **Step**'ler ise aynı job ve aynı runner içinde sırayla çalışır ve çalışma alanını paylaşır. "Bir step'te oluşan dosya diğer job'da neden yok?" sorusunun cevabı bu sınırdır; dosyayı başka bir job'a taşımak için artifact gerekir.

Job'lar bağımlılık belirtilmezse paralel başlar. `needs` ile bağımlılık kurulur ve bir bağımlılık grafiği oluşur: örneğin lint ve birim testleri paralel çalışır, build ikisinin de başarılı olmasını bekler, tarama build'den, dağıtım taramadan sonra gelir.

Üç tür runner vardır. **GitHub-hosted** runner'lar her job için temiz bir sanal makine sunar ve bakımı GitHub'dadır. **Larger runner**'lar daha fazla kaynak ve sabit IP gibi seçenekler sunar. **Self-hosted** runner'lar kurumun kendi donanımında ve ağında çalışır; iç sistemlere erişebilir ama yamalanması, ölçeklenmesi ve temiz tutulması kurumun sorumluluğudur. Bir job'ın bıraktığı dosyaların bir sonrakini etkilememesi için her job'dan sonra temizlenen geçici runner'lar idealdir.

**Matrix** stratejisi aynı job'ı birden çok varyasyonda çalıştırır; örneğin iki işletim sistemi ve iki runtime sürümüyle toplam dört paralel job. Farklı sürümleri desteklemesi gereken kütüphaneler için güçlüdür, ama job sayısı çarpılarak arttığı için maliyet ve ilk hatada durma davranışı bilinçli ayarlanmalıdır.

Dosya taşımanın iki farklı yolu vardır ve karıştırılmamalıdır. **Artifact**, bir çıktıyı sonraki job'a ya da bir insana vermek içindir; kaybolursa dağıtılacak paket yok demektir. **Cache** ise tekrar indirmeyi hızlandırmak içindir; kaybolursa yalnızca yavaşlanır. Bağımlılık önbelleği genellikle lockfile'ın özetiyle anahtarlanır.

## GitHub Actions ile CI/CD

İyi bir pull request pipeline'ı hızlı, deterministik ve tekrar üretilebilirdir: lint hızlı bir kapı olarak çalışır, ardından birim testleri, build ve güvenlik taraması gelir. Yalnızca okuma yetkisiyle başlar; yazma gerekiyorsa job düzeyinde artırılır. Kurulum, lint, test ve build ayrı step'ler olarak yazıldığında bir şey kırıldığında hangi adımın kırıldığı logda hemen görülür. Belirli dizinlerde değişiklik olduğunda çalışan path filtrelerine dikkat etmek gerekir: böyle bir workflow zorunlu bir kontrolse ve değişiklik o dizinlere dokunmuyorsa workflow hiç çalışmaz ve pull request beklemede kalır.

CI tek başına bir kapı değildir; onu birleştirme politikasına bağlamak gerekir. Ruleset ya da branch protection, workflow kontrollerini zorunlu tutar ve birleştirme yalnızca yeşilken mümkün olur. Zorunlu kontroller isimle eşlendiği için workflow ve job isimlerini sık değiştirmemek gerekir.

Container image üretimi Buildx ve BuildKit ile yapılır: kod çekilir, etiketler üretilir, build ve gönderim yapılır ve image'ın içerik özeti (digest) alınır. Registry kimlik doğrulaması build mantığından ayrı tutulur ve digest bir sonraki dağıtım job'ına çıktı olarak taşınır; dağıtım etiketle değil digest ile yapılır.

**Environment**'lar dağıtım hedeflerini tanımlar. Geliştirme ortamına otomatik dağıtım yapılabilir; staging entegrasyon testleri ve sürüm adayları için kullanılır; üretim ise zorunlu onaylayıcılar, branch ya da tag kısıtları ve yalnızca bu ortama ait secret'larla korunur. Koruma kuralları dağıtım job'ı başlamadan önce bir kapı oluşturur: build bir artifact üretir, staging'e dağıtılır, otomatik kalite ve güvenlik kapıları çalışır, bir onaylayıcı onay verir ve aynı artifact üretime gider. Bu, continuous delivery'deki onay adımının GitHub karşılığıdır.

## GitLab CI/CD

GitLab'da bir commit ya da merge request pipeline'ı tetikler, GitLab `.gitlab-ci.yml` dosyasını okuyup aşamaları ve job'ları çıkarır, uygun bir runner job'ı alıp çalıştırır ve loglar, artifact'ler ve durum GitLab'da görünür. Mantık GitHub Actions ile aynıdır; GitLab'ın farkı depo, registry, issue yönetimi, ortamlar ve güvenlik taramasının tek bir uygulamada olmasıdır.

GitLab'da dört seviye vardır: bir commit ya da merge request için bütün otomasyon olan **pipeline**; build, test ve deploy gibi sıralı fazlar olan **stage**'ler; runner'ın çalıştırdığı tek iş birimi olan **job**'lar ve job içindeki komut satırları olan **script**. Bir stage'deki bütün job'lar bitmeden sonraki stage başlamaz; bu bazen gereksiz bekleme yaratır. `needs` ile bu sıra kırılabilir: yalnızca derleme çıktısına bağlı bir entegrasyon testi, test aşamasındaki diğer job'ları beklemeden derleme biter bitmez başlayabilir.

Bir `.gitlab-ci.yml` dosyasında her job'ın ait olduğu stage, komutların çalışacağı container ortamını belirleyen image ve çalışacak script bulunur. GitLab'ın hazır değişkenleri, örneğin commit kimliği, image'ları izlenebilir biçimde etiketlemeyi sağlar. Job'ların ne zaman çalışacağı `rules` ile yazılır; örneğin üretim dağıtımı bir sürüm tag'iyle tetiklenir ve `when: manual` ile insan onayı bekler. Eski tanımlarda görülen only ve except yerine yeni tanımlarda rules kullanılır. İki seviyeli kurallar da vardır: workflow kuralları bütün pipeline'ın oluşturulup oluşturulmayacağını, job kuralları ise hangi job'ların pipeline'a gireceğini belirler. Workflow kuralları, aynı commit için hem branch hem merge request pipeline'ı çalışması sorununu da çözer.

GitHub Actions bilen biri için iki araç şöyle eşlenir:

| Kavram | GitHub Actions | GitLab CI |
|---|---|---|
| Pipeline dosyası | .github/workflows/*.yml | .gitlab-ci.yml |
| Sıralama | needs | stages ve needs |
| Koşul | on ve if | workflow rules ve rules |
| Ortam ve onay | environment ve onaylayıcı | protected environment, manual |
| Yeniden kullanım | reusable workflow | include, extends, components |
| Paralel dağıtım kilidi | concurrency | resource_group |
| Bulut kimliği | OIDC | OIDC |

Birçok kurum GitLab'ı kendi sunucusuna kurar. Bunun sebepleri verinin kurumun seçtiği yerde kalması, kapalı ağlarda kurum içi servislerle çalışabilmesi, LDAP, tek oturum açma ve denetim gereksinimlerinin bağlanabilmesi ve sürüm ile bakım kararlarının kurumda olmasıdır. Ama kontrol kurumdayken bakım, kapasite, yedek ve güvenlik sorumluluğu da kurumdadır. Kurum içi GitLab'ın parçaları web arayüzü ve API'yi sunan uygulama katmanı, arka plan işleri, CI job'larını çalıştıran runner'lar ve durum tutan veri katmanıdır: metadata için PostgreSQL, önbellek ve kuyruk için Redis, Git verisi için Gitaly ve artifact'ler ile yedekler için nesne depolama. Yedek planı bu dört bileşeni kapsamalıdır; ayrıca GitLab'ın gizli anahtar dosyası yedeklenmezse geri yüklemeden sonra CI değişkenleri çözülemez. Güncellemelerde sürüm atlanmamalı, GitLab'ın önerdiği güncelleme yolu izlenmelidir.

Runner kaydolur, job'lar kuyrukta bekler, GitLab job ile runner'ı tag'e, executor'a ve kapasiteye göre eşleştirir, runner job'ı çalıştırır ve sonuç geri gelir. En yaygın executor Docker'dır: runner her job için seçilen image'dan bir container başlatır ve script orada çalışır; yanındaki yardımcı container kodu çekme, artifact ve önbellek işlerini yapar. Job image'ının bir çalışma ortamı olduğu, uygulama image'ının ise ayrıca üretildiği unutulmamalıdır.

GitLab'da bir değişken birçok yerde tanımlanabilir ve aynı ad birden çok yerdeyse bir öncelik sırası uygulanır: pipeline çalıştırılırken verilen değişkenler en yüksek, GitLab'ın hazır değişkenleri en düşük önceliğe sahiptir; arada proje, grup ve dosya düzeyindeki değişkenler bulunur. "Değişkenim neden beklediğim değeri almıyor?" sorusunun cevabı genellikle bu sıradadır. Secret değişkenler maskelenmeli ve korumalı olarak işaretlenmelidir.

GitLab her dağıtımı bir ortama bağlı kayıt olarak tutar: hangi commit'in ne zaman ve kim tarafından dağıtıldığı görülebilir ve eski başarılı bir dağıtım tek tıkla yeniden çalıştırılabilir. Korumalı ortamlar üretime yalnızca yetkili kişilerin dağıtım yapmasını sağlar. Her merge request için geçici bir ortam açan review app'ler, inceleyenlerin değişikliği canlı görmesini sağlar ve belirli bir süre sonra kendiliğinden kapanır. resource_group ile aynı ortama paralel dağıtım engellenir. Bir Docker sunucusuna dağıtım yapılırken komutların hedef sunucuda çalıştığından emin olmak gerekir; CI job'ının içindeki geçici Docker'a dağıtım yapmak kalıcı sunucuyu güncellemez.

## Sık yapılan hatalar

GitHub Actions'ta sık yapılan hatalar şunlardır: workflow'a bütün yazma yetkilerini vermek, eylemleri bir branch adıyla kullanmak, güvenilmeyen pull request kodunu secret'lara erişimi olan bir bağlamda çalıştırmak, event verisini doğrudan komut satırına yerleştirerek script enjeksiyonuna kapı açmak, uzun ömürlü bulut anahtarları kullanmak, testlerde hatayı yok sayan ayarlar kullanmak, her ortam için yeniden build etmek ve üretim dağıtımlarında eşzamanlılık kilidi kullanmamak. Bu hataların çoğu actionlint ve zizmor gibi araçlarla otomatik bulunabilir.

GitLab'da ise yeni pipeline'larda only ve except kullanmak, branch ve merge request için çift pipeline çalıştırmak, ayrıcalıklı paylaşımlı runner'larda Docker-in-Docker kullanmak, secret'ları maskelememek, gizli anahtar dosyasını yedeklememek, sürüm atlayarak güncellemek, latest etiketiyle dağıtım yapmak ve üretimde dağıtım kilidi kullanmamak sık görülen hatalardır.

> GitHub Actions'ta event workflow'u, workflow job'ları, job'lar step'leri çalıştırır; job'lar ayrı makinelerdedir ve dosyalar artifact ile taşınır. GitLab CI pipeline, stage, job ve script seviyeleriyle aynı fikri uygular. İki araçta da en az yetki, commit kimliğiyle sabitleme, environment korumaları ve OIDC güvenli bir pipeline'ın temelidir.
