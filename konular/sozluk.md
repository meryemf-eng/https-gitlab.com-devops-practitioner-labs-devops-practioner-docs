# Sözlük

Kitapta geçen temel terimler, alfabetik sırayla ve kısa tanımlarıyla.

| Terim | Tanım |
|---|---|
| Admission kontrolü | Kubernetes API server'a gelen isteği kayıttan önce değiştiren ya da reddeden kontrol katmanı |
| Alertmanager | Prometheus'tan gelen alarmları gruplayan, yönlendiren, bastıran ve susturan bileşen |
| Ansible | Agent gerektirmeden SSH ile sunucu yapılandırmasını yöneten otomasyon aracı |
| Argo CD | Kubernetes için pull tabanlı, declarative GitOps teslimat aracı |
| Artifact | Build sonucunda üretilen, dağıtılabilir çıktı: paket, image ya da chart |
| Blue-green | İki eş ortam arasında trafiği bir anda yeni sürüme çeviren dağıtım stratejisi |
| Branch protection | Bir branch'e doğrudan gönderimi engelleyip inceleme ve kontrol şartı koyan kural |
| Burn rate | Hata bütçesinin normalden kaç kat hızlı tükendiğini gösteren oran |
| CALMS | Kültür, otomasyon, yalın, ölçüm ve paylaşımdan oluşan DevOps çerçevesi |
| Canary | Yeni sürümü önce trafiğin küçük bir kısmına açan kademeli dağıtım stratejisi |
| Cardinality (kardinalite) | Bir metrikteki benzersiz etiket kombinasyonu, yani zaman serisi sayısı |
| CI | Sürekli entegrasyon: her değişikliğin otomatik derlenip test edilmesi |
| CD | Sürekli teslimat ya da sürekli dağıtım: doğrulanmış değişikliğin otomatik olarak yayına hazırlanması ya da yayınlanması |
| ConfigMap | Kubernetes'te gizli olmayan yapılandırmayı tutan nesne |
| Container | Uygulamayı bağımlılıklarıyla birlikte izole eden, çekirdeği paylaşan süreç |
| Cosign | Container image'larını imzalayan ve doğrulayan araç |
| Counter | Yalnızca artan, yeniden başlatmada sıfırlanan metrik tipi |
| CVE | Bilinen bir güvenlik açığına verilen standart kimlik |
| DaemonSet | Her node'da bir Pod çalıştıran Kubernetes nesnesi |
| Data stream | Elasticsearch'te zaman tabanlı verileri arka planda index'lere bölen yapı |
| Declarative | Adımları değil, olması gereken son durumu tarif eden yaklaşım |
| Deployment | Kubernetes'te kopya sayısını ve sürüm geçişini yöneten nesne |
| DevSecOps | Güvenliği teslimat akışının otomatik ve ortak sorumluluk olan bir parçası yapmak |
| Digest | Bir image'ın içeriğinden hesaplanan, değişmez kimlik |
| Docker Compose | Birden çok container'ı tek bir dosyayla tanımlayıp çalıştıran araç |
| DORA metrikleri | Dağıtım sıklığı, değişiklik teslim süresi, değişiklik başarısızlık oranı ve geri yükleme süresi |
| Drift | Gerçek durumun koddaki istenen durumdan sapması |
| ECR | AWS'nin yönetilen container registry servisi |
| EKS | AWS'nin yönetilen Kubernetes servisi |
| Elasticsearch | Dağıtık arama ve analiz motoru |
| Error budget (hata bütçesi) | Hizmet hedefinin izin verdiği hata payı |
| Exporter | Kendisi metrik sunmayan bir sistemin metriklerini Prometheus biçiminde sunan bileşen |
| Feature flag | Bir özelliği dağıtımdan bağımsız olarak açıp kapatan anahtar |
| Filebeat | Dosya ve container loglarını toplayan hafif ajan |
| Gauge | Artıp azalabilen metrik tipi |
| Game day | Sistemi bilerek bozup müdahale pratiği yapılan tatbikat |
| Git | Dağıtık sürüm kontrol sistemi |
| GitHub Actions | GitHub'ın olay tetiklemeli CI/CD platformu |
| GitLab CI | GitLab'ın depo içi tanımlı CI/CD sistemi |
| GitOps | Git'i istenen durumun kaynağı yapıp gerçek durumu sürekli onunla uzlaştıran model |
| Golden path | Platform ekibinin sunduğu, önerilen ve desteklenen standart yol |
| Grafana | Birden çok veri kaynağını görselleştiren pano ve analiz platformu |
| Grok | Logstash'te düz metin satırlarını desenlerle alanlara ayıran filtre |
| Harbor | Tarama, imza, çoğaltma ve rol tabanlı erişim sunan kurumsal registry |
| Helm | Kubernetes için paket yöneticisi; chart'lar ve release'ler |
| Histogram | Değerleri kovalara dağıtarak yüzdelik hesaplamayı sağlayan metrik tipi |
| HPA | Kubernetes'te Pod sayısını metriklere göre otomatik ölçekleyen nesne |
| IaC | Altyapıyı makinece işlenebilir, sürümlenen kodla yönetmek |
| Idempotent | Tekrar çalıştırıldığında aynı sonucu veren, gereksiz değişiklik yapmayan işlem |
| ILM | Elasticsearch'te index'leri hot, warm, cold ve delete fazlarında yöneten yaşam döngüsü |
| Image | Container'ın çalıştırıldığı, katmanlardan oluşan değişmez şablon |
| Immutable infrastructure | Sunucuyu değiştirmek yerine yenisini üretip eskisini atan yaklaşım |
| Ingress | Kubernetes'te HTTP trafiğini servislere yönlendiren kurallar |
| Jenkins | Eklentilerle genişleyen, kendi sunucusunda çalışan CI/CD sunucusu |
| Jenkinsfile | Jenkins pipeline'ını kod olarak tanımlayan dosya |
| Kibana | Elasticsearch için arama, pano ve yönetim arayüzü |
| KQL | Kibana'nın filtreleme sorgu dili |
| kubectl | Kubernetes'in komut satırı aracı |
| Kubernetes | Container'ları cluster üzerinde planlayan ve istenen durumda tutan orkestrasyon platformu |
| Kyverno | Kubernetes için YAML tabanlı politika motoru |
| Lead time | Bir değişikliğin commit'ten üretime ulaşma süresi |
| LLM | Bir sonraki token'ı olasılıkla tahmin ederek metin üreten büyük dil modeli |
| Lockfile | Bağımlılıkların tam sürümlerini sabitleyen dosya |
| Logstash | Girdi, filtre ve çıktı aşamalarından oluşan log işleme hattı |
| MCP | Yapay zekâ ajanlarını araçlara bağlayan ortak protokol |
| Merge request / pull request | Bir değişikliğin incelenip ana branch'e alınması için açılan istek |
| MTTR | Bir olaydan sonra hizmetin geri gelmesine kadar geçen ortalama süre |
| Namespace | Kubernetes kaynaklarını mantıksal olarak ayıran alan |
| Observability | Sistemin ürettiği telemetriden iç durumunu anlayabilme yeteneği |
| OIDC | Kısa ömürlü kimlik belirteçleriyle kimlik doğrulama standardı |
| OpenTelemetry | Metrik, log ve trace için üreticiden bağımsız standart ve araç seti |
| OWASP | Uygulama güvenliği için açık topluluk ve risk listeleri |
| Persistent volume | Kubernetes'te Pod'dan bağımsız yaşayan depolama |
| Pipeline | Değişikliği derleyen, test eden ve dağıtan otomatik adımlar zinciri |
| Platform engineering | Geliştiricilere self-servis iç platform sunan disiplin |
| Pod | Kubernetes'te birlikte planlanan bir ya da daha çok container'dan oluşan en küçük birim |
| Postmortem | Bir olaydan sonra suçlamasız biçimde yazılan inceleme ve aksiyon raporu |
| Probe | Kubernetes'te container'ın canlılık, hazırlık ve başlangıç kontrolü |
| Prometheus | Pull modeliyle zaman serisi metrikleri toplayan izleme sistemi |
| PromQL | Prometheus'un sorgu dili |
| Prompt injection | Modelin okuduğu içeriğe gizlenmiş talimatlarla davranışını ele geçirme girişimi |
| RAG | Soruya göre belgeleri arayıp model yanıtına ekleme yöntemi |
| RBAC | Rol tabanlı erişim kontrolü |
| Reconciliation | İstenen durum ile gerçek durumu karşılaştırıp farkı kapatma döngüsü |
| RED | İstek hızı, hata ve süreden oluşan servis ölçüm yöntemi |
| Registry | Container image'larının saklandığı ve dağıtıldığı depo |
| Replica | Bir Pod'un ya da shard'ın kopyası |
| Rollback | Önceki çalışan sürüme geri dönme |
| Rolling update | Pod'ları kademeli olarak yeni sürümle değiştiren dağıtım |
| Runbook | Bir alarm ya da olayda yapılacakları anlatan müdahale rehberi |
| Runner / agent | Pipeline job'larını çalıştıran makine ya da süreç |
| SAST | Kaynak kodu çalıştırmadan analiz eden güvenlik testi |
| SBOM | Bir yazılımın içerdiği bileşenlerin listesi |
| SCA | Bağımlılıkların bilinen açıklarını ve lisanslarını kontrol eden analiz |
| Secret | Parola, token ve anahtar gibi hassas bilgi; Kubernetes'te bunları tutan nesne |
| Self-healing | Sapmanın ya da hatanın otomatik olarak düzeltilmesi |
| Service | Kubernetes'te Pod grubuna sabit bir ağ adresi veren nesne |
| Shard | Elasticsearch index'inin parçası |
| Shift-left | Test ve güvenlik kontrollerini yaşam döngüsünde öne çekmek |
| SLA | Müşteriye taahhüt edilen ve yaptırımı olan hizmet seviyesi |
| SLI | Hizmet seviyesini ölçen gösterge |
| SLO | Bir SLI için belirlenen iç hedef |
| SonarQube | Kod kalitesi ve güvenlik analizi platformu |
| SRE | Yazılım mühendisliği yöntemleriyle güvenilirlik sağlama disiplini |
| State (Terraform) | Terraform'un yapılandırma ile gerçek kaynaklar arasındaki eşleşmeyi tuttuğu kayıt |
| StatefulSet | Kubernetes'te kalıcı kimlik ve depolama gerektiren uygulamalar için nesne |
| Tag (etiket) | Bir image ya da commit için okunabilir ve değişebilir ad |
| Terraform | Declarative altyapı kodu aracı |
| Three Ways | Akış, geri bildirim ve sürekli öğrenmeden oluşan DevOps ilkeleri |
| Toil | Elle yapılan, tekrarlayan ve otomatikleştirilebilir operasyon işi |
| Trace / span | Bir isteğin servisler arasındaki yolculuğu ve bu yolculuktaki tek bir işlem |
| Trivy | Image, dosya sistemi, altyapı kodu ve cluster tarayan güvenlik aracı |
| USE | Kullanım, doygunluk ve hatadan oluşan kaynak ölçüm yöntemi |
| VPC | Bulutta izole sanal ağ |
| Webhook | Bir olay olduğunda başka bir sisteme gönderilen HTTP çağrısı |
| YAML | Kubernetes, CI/CD ve Ansible tanımlarında kullanılan, girintiye dayalı veri biçimi |
