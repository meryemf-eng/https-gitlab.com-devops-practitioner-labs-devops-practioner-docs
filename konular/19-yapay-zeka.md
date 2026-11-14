# Yapay zekâ ve DevOps

Yapay zekâ araçları DevOps mühendisinin günlük işine hızla girdi: Dockerfile taslağı yazmak, kırmızı bir pipeline'ın logunu okumak, Terraform planını özetlemek, bir alarm kuralı üretmek. Bu doküman araç tanıtımı değil, bir çalışma modelidir. Büyük dil modellerinin nasıl çalıştığını ve neden yanıldığını, yapay zekâ çıktısının DevOps araçlarıyla nasıl doğrulandığını, ajanları ve araç bağlantılarını, operasyonda yapay zekâyı ve güvenlik ile sorumluluk sınırlarını ele alıyoruz.

## Temel kavramlar

Yapay zekâ terimleri iç içe geçen halkalar gibidir.

**Yapay zekâ**, insan zekâsı gerektiren işleri yapan sistemlerin genel adıdır. **Makine öğrenmesi**, kuralı elle yazmak yerine veriden öğrenen modellerdir. **Derin öğrenme**, çok katmanlı sinir ağlarıyla yapılan makine öğrenmesidir. **Üretken yapay zekâ** ve onun en bilinen örneği olan **büyük dil modelleri** (LLM) ise metin, kod ve görsel üretir. DevOps'ta iki farklı kullanım vardır. Klasik makine öğrenmesi, anomali tespiti, kapasite tahmini ve alarm gürültüsünü azaltma gibi işlerde yıllardır izleme platformlarının içinde çalışır. LLM'ler ise kod, YAML ve sorgu üretir, log ve hata mesajlarını açıklar. Bir **yapay zekâ asistanı** sohbet ederek öneri sunar; bir **yapay zekâ ajanı** ise araç çağırarak çok adımlı bir görevi kendisi yürütür.

Bir LLM'in nasıl çalıştığını anlamak, onu doğru kullanmanın ön koşuludur.

Girdi, yani prompt ve bağlam, **token** denen parçalara bölünür. Model bir sonraki token için olasılıkları hesaplar, birini seçer ve yanıt bu şekilde parça parça oluşur. Buradan altı önemli sonuç çıkar. Ücret ve sınırlar token ile ölçülür. Modelin tek seferde gördüğü metin miktarı, yani **bağlam penceresi**, sınırlıdır ve dışındaki bilgiyi bilmez. Model belirli bir tarihe kadarki veriyle eğitilmiştir ve güncel sürümleri bilmeyebilir. En olası cevap her zaman doğru cevap değildir: "bilmek" ile "uydurmak" aynı mekanizmadır ve **halüsinasyon** buradan doğar. Model arama, dosya okuma ve komut çalıştırma gibi araçlarla güncel bilgiye erişebilir. Son olarak model deterministik değildir: aynı soru farklı zamanlarda farklı cevap verebilir. Bu yüzden her çıktı doğrulanmalıdır.

Modelden iyi sonuç almanın dört yolu vardır ve maliyetleri farklıdır. **Prompt**, görevi açık, örnekli ve kısıtlı yazmaktır; iyi bir prompt hedefi, ortamı, kısıtları, beklenen çıktı biçimini ve doğrulama yöntemini içerir. "Bir Dockerfile yaz" yerine "Node 22 için, root olmayan kullanıcıyla, çok aşamalı ve yalnızca üretim bağımlılıklarını kuran bir Dockerfile yaz; hadolint ve image taraması temiz olsun" demek çok daha iyi sonuç verir. **Bağlam**, ilgili dosyaları, logları ve kuralları modele vermektir; depodaki bir bağlam dosyası, örneğin bir AGENTS.md, ekibin kurallarını her oturumda modele aktarır. **RAG** (retrieval-augmented generation), soruya göre runbook ve postmortem arşivi gibi belgeleri arayıp ilgili parçaları yanıta eklemektir. **Fine-tuning** ise modeli kendi verinle yeniden eğitmektir ve maliyetlidir. DevOps işlerinin büyük çoğunluğu ilk üç yöntemle çözülür.

## Modeller ve araçlar

Genel amaçlı asistanlar benzer işleri yapar; aralarındaki fark bağlam büyüklüğü, araç entegrasyonu ve kurumsal koşullardadır. Bunların yanında, kendi sunucunda çalıştırılabilen açık modeller de vardır ve bunlarda veri kurum dışına çıkmaz. Kurumda hangi aracın kullanılacağı teknik bir karar değil, bir **veri politikası** kararıdır: kod ve loglar nereye gidiyor, saklanıyor mu, eğitimde kullanılıyor mu?

Kodlama araçları üç kuşakta gelişti: satır içi tamamlama, sohbet ve ajan. Tamamlama bir sonraki satırı önerir; sohbet bir soruya kod ile cevap verir; ajan ise depoyu okur, birden çok dosyayı değiştirir, komut çalıştırır ve sonucu değerlendirerek görevi tamamlar. Yetki arttıkça doğrulama ihtiyacı da artar. Bütün kodlama araçları için ortak kural şudur: üretilen kod, insanın yazdığı kodla aynı pipeline'dan geçer; lint, test, tarama ve inceleme atlanmaz.

Yapay zekâya ne kadar iş bırakılacağı beş seviyeyle düşünülebilir. Birinci seviyede yapay zekâ yalnızca öneri yapar ve insan kopyalayıp uygular. İkinci seviyede dosyaları değiştirir ve insan diff'i inceleyip onaylar. Üçüncü seviyede her biri onaylanan komutları çalıştırır; örneğin bir helm lint ya da terraform plan. Dördüncü seviyede bir sandbox içinde kendi döngüsünü çalıştırır ve insan sonucu ve açılan pull request'i inceler. Beşinci seviye, canlı sisteme doğrudan yazma yetkisidir ve önerilmez. Temel kural: yapay zekânın yetkisi, onu çalıştıran kişinin yetkisinden fazla olamaz ve üretime yazma yetkisi pipeline'dadır.

## DevOps'ta kullanım alanları

Yapay zekâ en çok "ilk taslak" ve "ilk analiz" adımlarında zaman kazandırır. Sekiz tipik kullanım alanı vardır: Bash, Python, Dockerfile ve Helm şablonu gibi **kod ve betik** taslakları; GitHub Actions, GitLab CI ve Jenkinsfile gibi **pipeline** tanımlarını üretmek ve hatalı adımı bulmak; Terraform modülü yazmak ve plan çıktısını özetleyip riskli değişikliği işaretlemek; birim ve smoke **testleri** üretmek; uzun **logları** özetleyip tekrar eden hataları gruplamak; bir hata mesajından olası nedenler ve kontrol listesi çıkarmak; README, runbook ve postmortem gibi **dokümantasyon** taslakları; secret, açık port, root container ve geniş yetki gibi sorunlar için **güvenlik incelemesi**. Her durumda yapay zekâ taslağı yazar; doğruluğunu araçlar ve insan kanıtlar.

Yapay zekâ ile güvenli çalışmanın beş adımlı bir döngüsü vardır.

Önce **bağlam verilir**: depo, sürümler, kısıtlar ve örnekler. Sonra küçük ve tek amaçlı bir değişiklik **üretilir**. Ardından lint, test, tarama ve kuru çalıştırma ile **doğrulanır**. Sonra diff okunur ve gerekirse nedeni sorulur, yani **incelenir**. En son pull request, pipeline ve GitOps yoluyla **uygulanır**. Döngüye ihtiyaç duyulmasının sebebi, modelin akıcı ama yanlış olabilmesidir: var olmayan bir parametre, kaldırılmış bir API sürümü ya da uydurma bir paket önerebilir. Hız, doğrulama döngüsü kadar güvenlidir. Bir de sahiplik meselesi vardır: kodu birleştiren kişi kodun sahibidir; "yapay zekâ yazdı" bir gerekçe değildir.

DevOps araçları, yapay zekâ çıktısının test takımıdır:

| Yapay zekâ ne üretir? | Nasıl doğrulanır? |
|---|---|
| Dockerfile | hadolint, docker build, image taraması, çalıştırıp sağlık kontrolü |
| Kubernetes manifest | kubeconform, sunucu taraflı kuru çalıştırma, Kyverno |
| Helm chart | helm lint, helm template, chart testleri |
| GitHub Actions ve GitLab CI | actionlint, GitLab CI doğrulayıcı, zizmor |
| Jenkinsfile | Declarative linter, replay |
| Terraform | fmt, validate, tflint, Trivy, plan |
| Ansible | ansible-lint, kuru çalıştırma ve diff, Molecule |
| PromQL ve alarm kuralları | promtool ile kural kontrolü ve kural testleri |

Bu eşleşmenin pratikteki anlamı birkaç örnekle görülür. Bir **Dockerfile** isteğinde yapay zekânın tipik hataları latest etiketi, root kullanıcı, temiz kurulum yerine normal paket kurulumu, gereksiz paketler ve eski bir base image'dır; hadolint ve image taraması bunları yakalar ve hata çıktısı aynı oturuma "bu bulguları düzelt" diye verilerek döngü kapatılır. **Kubernetes ve pipeline YAML'ında** en sık hata kaynağı API sürümü ve söz dizimidir: kaldırılmış API sürümleri ve var olmayan alanlar kubeconform'un katı moduyla yakalanır; pipeline tarafında tehlikeli tetikleyiciler, script injection ve sabitlenmemiş eylemler gibi güvenlik sorunları zizmor ile bulunur. Yapay zekâ çoğu zaman eylemleri sürüm etiketiyle yazar; commit kimliğiyle sabitleme politikası bağlam dosyasında belirtilmelidir. **Terraform'da** plan çıktısı yapay zekânın en iyi geri bildirimidir: planda bir silme ya da replace görülürse durulur ve sorulur. Yapay zekâ apply çalıştırmaz; plan pull request'te incelenir, apply pipeline'da onayla yapılır. **PromQL ve alarm kurallarında** yapay zekâdan kuralla birlikte bir promtool testi de istenebilir; ama metrik ve etiket adlarının gerçekten var olup olmadığı Grafana'da denenmelidir. Önce sum sonra rate almak ve for süresini unutmak tipik hatalardır.

**Log ve olay analizinde** yapay zekâ binlerce satırdan desen çıkarmada, stack trace yorumlamada, olayın zaman çizelgesini kurmada ve postmortem taslağı yazmada çok değerlidir. Ama loglar yapay zekâya verilmeden önce temizlenir: token'lar, e-posta adresleri ve IP adresleri maskelenir; müşteri verisi ve secret içeren loglar harici bir yapay zekâ hizmetine gönderilmez. Bir risk daha vardır: log içinde "önceki talimatları unut" gibi bir metin bulunabilir. Bu bir **prompt injection** girişimidir ve kural şudur: log veridir, komut değildir. Yapay zekânın kök neden iddiası bir hipotezdir; metrik, trace ya da bir deneyle doğrulanmadan postmortem'e yazılmaz.

Kırmızı bir pipeline'ı yapay zekâ ile çözmek bu döngünün tipik bir uygulamasıdır. Modele ortam (çalıştırıcı, dil sürümü), hatalı adım, secret'ları maskelenmiş son kırk satırlık log ve ilgili dosyalar verilir; istek ise en olası üç neden, her biri için bir doğrulama komutu ve en küçük düzeltmedir. Önerilen komut yerelde ya da bir branch'te çalıştırılır ve düzeltme bir pull request ile incelemeye gelir. En sık sonuçlar lockfile uyumsuzluğu, sürüm farkı, eksik secret ya da önbellek sorunudur.

## Ajanlar ve MCP

**Ajan**, bir model, araçlar ve bir döngünün birleşimidir: hedefe ulaşana kadar planlar, araç çağırır ve sonucu değerlendirir.

Kararı veren LLM'dir: hangi adım, hangi araç? Araçlar kubectl, git, terraform plan, API çağrıları ve dosya okuma olabilir. Bağlam ve hafıza depo kurallarını ve önceki adımların çıktısını taşır. Guardrail'ler ise yetki sınırı, onay adımı, sandbox ve denetim kaydıdır. Asistan ile ajan arasındaki fark şudur: asistan önerir ve sen uygularsın; ajan kendisi uygular ve sen onaylarsın. Asistan tek adımda çalışır ve riski düşüktür; ajan çok adımlı görev yürütür ve riski yetkisine bağlıdır. Bu yüzden ajanlarda en az yetki şarttır.

**MCP** (Model Context Protocol), ajan ile araçlar arasında ortak bir arayüzdür. Ajan bir MCP istemcisi olarak GitHub, Kubernetes, Grafana ya da Terraform gibi araçların MCP sunucularına bağlanır ve onların sunduğu işlemleri çağırır. Her bağlantı için önerilen yetki farklıdır: bir Git platformu sunucusu issue, pull request ve pipeline sonuçlarını okuyabilir, pull request açması ise onaya bağlıdır; Kubernetes sunucusu yalnızca okuma rolüyle kaynak, log ve olay listeleyebilir; Grafana sunucusu izleyici yetkisiyle pano ve alarm durumuna bakabilir; dosya sistemi ve kabuk erişimi sandbox içinde ve onaylı komutlarla verilir. MCP sunucusu da bir **bağımlılıktır**: resmi ya da doğrulanmış kaynaklardan alınmalı, sürümü sabitlenmeli ve yetkisi daraltılmalıdır, çünkü bir ajana araç eklemek ona yeni bir yetki vermektir.

Yapay zekâ pipeline'ın içinde de kullanılabilir; en yaygın örnek, her pull request'te kod incelemesi yapıp yorum bırakan bir job'dır. Bu job'ın kuralları bellidir: yalnızca okuma ve yorum yazma yetkisi taşır, onay veremez ve birleştiremez, insan incelemesi yine şarttır. Fork'tan gelen pull request'lerde secret'a erişemez. Mantık hataları, eksik testler, güvenlik kokuları ve eksik dokümantasyon arar. Yorumların ne kadarının kabul edildiği ölçülür; gürültü üretiyorsa kapatılır.

Ajanlar için guardrail'ler, insanlar için kurduğumuz kapılarla aynıdır: üretime yazma yetkisi olmayan ayrı ve salt okunur bir kimlik; ağ erişimi kısıtlı bir container ya da geçici ortam; silme, apply, destroy ve zorla gönderim gibi yıkıcı komutlar için onay; önce plan ve kuru çalıştırma; ajanın çalıştırdığı her komutun ve araç çağrısının kaydı; tek depo, tek görev, süre ve bütçe sınırı. Pipeline kapıları ve admission politikası ajanın hatasını da yakalar.

## AIOps ve bulutta yapay zekâ

**AIOps**, binlerce sinyal arasından önemli olanı bulmak için operasyonda yapay zekâ kullanmaktır. Anomali tespiti, eşik yazmadan normal davranışı öğrenip sapmayı yakalar. Olay korelasyonu, aynı kök nedenden gelen yüzlerce alarmı tek bir olayda toplar. Gürültü azaltma, tekrarlayan ve önemsiz alarmları bastırır. Kök neden önerisi metrik, log, trace ve dağıtım kayıtlarını ilişkilendirir. Tahmin, disk dolma, kapasite ve maliyet eğilimlerini öngörür. Olay özeti ise insan için bir zaman çizelgesi ve postmortem taslağı üretir. AIOps iyi telemetri ister: kötü etiketlenmiş metrikler ve yapılandırılmamış loglarla yapay zekâ da kör kalır. Önceki iki konuda anlatılan etiket disiplini ve yapılandırılmış log, burada doğrudan karşılığını bulur.

İzleme platformları artık alarmı göstermekle kalmıyor, ilk incelemeyi kendisi yapıyor: bir alarm geldiğinde telemetriye ve runbook'lara bakıp olası kök nedeni öneren ajanlar, doğal dilden PromQL, LogQL ya da ES|QL sorgusu üreten asistanlar ve cluster sorunlarını açıklayan açık kaynak araçlar. Bulut sağlayıcıları da operasyon ajanlarını servis olarak sunmaya başladı. Ürün adları hızla değişir, ama kavram aynıdır: telemetri, bağlam ve bir LLM birleşerek bir kök neden önerisi üretir. Ortak kalıp da aynıdır: ajan okur ve önerir, değişikliği onaylı pipeline uygular. Bir ajana üretim erişimi vermeden önce dört soru sorulur: hangi rol, hangi kaynak, hangi onay ve hangi kayıt?

## Güvenlik ve sorumluluk

Yapay zekânın DevOps'a özgü riskleri vardır. **Veri sızıntısı**: bir .env dosyasını ya da kubeconfig'i sohbete yapıştırmak. **Prompt injection**: issue, README ya da log içine gömülmüş talimatlar. **Uydurma paket**: modelin var olmayan bir paketi önermesi ve saldırganların bu adla zararlı paketler yayımlaması; lockfile, iç registry ve bağımlılık taraması buna karşı korur. **Güvensiz varsayılanlar**: ayrıcalıklı container, tüm internete açık kurallar, root kullanıcı. **Aşırı yetki**: cluster yöneticisi kimliğiyle çalışan bir ajan. **Lisans**: lisanslı bir kod parçasının kopyalanması.

OWASP, LLM uygulamaları için ayrı bir risk listesi yayımlar ve yapay zekâ kullanan her sistemin tehdit modeli bu başlıklarla başlar:

| Risk | DevOps'taki karşılığı |
|---|---|
| Prompt injection | Log, issue ya da README içine gizlenmiş talimat |
| Hassas bilgi ifşası | Prompt'a yapıştırılan secret ve müşteri verisi |
| Tedarik zinciri | Doğrulanmamış model, eklenti ya da MCP sunucusu |
| Veri ve model zehirlenmesi | Bozuk eğitim ya da RAG verisi |
| Güvensiz çıktı işleme | Üretilen komutu kontrolsüz çalıştırmak |
| Aşırı yetki | Ajana cluster yöneticisi yetkisi vermek |
| Sistem prompt sızıntısı | Gizli kuralların ve anahtarların açığa çıkması |
| Vektör ve embedding zafiyetleri | RAG deposunda yetkisiz erişim |
| Yanlış bilgi | Var olmayan paket ya da parametre önermesi |
| Sınırsız tüketim | Kontrolsüz token kullanımı ve maliyet |

Demoda etkileyici olan, üretimde sorumluluk ister. Halüsinasyon, tekrarlanamazlık, veri sızıntısı, prompt injection ve aşırı yetkinin yanında token maliyeti ve yanıt süresinin kritik yolda sorun olması ve sağlayıcının modeli güncellediğinde davranışın değişmesi de üretim sorunlarıdır. Ve her zaman, hatalı değişikliğin sahibi bir insandır.

Bu yüzden risk arttıkça yapay zekânın rolü "yapan"dan "öneren"e iner.

Yapay zekâ analiz eder ve önerir, insan inceler, pipeline doğrular, değişiklik onayla uygulanır ve izlenip gerekirse geri alınır. Dokümantasyon ve test taslağı gibi düşük riskli işlerde yapay zekâ yazar ve insan göz atar. Kod, Dockerfile ve Terraform gibi orta riskli işlerde yapay zekâ önerir ve pull request açar, inceleme ve birleştirme kararı insandadır. Üretim dağıtımı, kimlik yetkileri ve ağ kuralları gibi yüksek riskli işlerde yapay zekâ yalnızca analiz ve öneri yapar, onay zorunludur ve iki kişi kuralı uygulanır. Olay anında geri alma ya da veri silme gibi kritik işlerde yapay zekâ zaman çizelgesi ve hipotez sunar, kararı veren ve uygulayan insandır. Yapay zekâ hızlandırır; hesap verebilirlik devredilmez.

Yapay zekânın etkisi hissedilen hızla değil, teslimat sonuçlarıyla ölçülür. DORA metrikleri teslimatın gerçekten hızlanıp hızlanmadığını ve hızın kaliteyi bozup bozmadığını gösterir. Pull request inceleme süresi ve boyutu, büyük yapay zekâ diff'lerinin incelemeyi yavaşlatıp yavaşlatmadığını; geri alma ve yeniden çalışma oranı, yapay zekâ kodunun daha sık geri alınıp alınmadığını; güvenlik bulgularının sayısı, tarama bulgularının artıp artmadığını gösterir. Geliştirici memnuniyeti de anketlerle izlenir. DORA araştırmalarının vardığı sonuç şudur: yapay zekâ bir yükselticidir; güçlü pratikleri hızlandırır, zayıf olanları da büyütür.

Kaçınılması gereken hatalar bu dokümanın özeti gibidir: çıktıyı okumadan birleştirmek, tek ve dev bir istekle bütün sistemi yazdırmak, secret'ı sohbete yapıştırmak, ajana cluster yöneticisi yetkisi vermek, "yapay zekâ dedi" diye bir kök neden yazmak, doğrulama araçlarını atlamak, bağlam dosyası olmadan çalışmak ve ölçmeden yaygınlaştırmak.

> Yapay zekâ bir araçtır: hızlandırır, ama doğrulamanın ve kararın yerine geçmez. LLM olasılıkla çalışır ve akıcı cevap doğru cevap değildir; bağlam kaliteyi belirler; ajan yetki demektir ve en az yetki, sandbox, onay ve denetim kaydı olmadan çalışmaz. Secret ve müşteri verisi prompt'a girmez ve üretim değişikliğinin sorumlusu her zaman bir insandır.
