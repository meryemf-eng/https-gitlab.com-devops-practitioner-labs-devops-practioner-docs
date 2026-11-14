# DevOps Practitioner: konu özeti

Her konu için kısa bir giriş ve dokümanın başlıkları. Ayrıntı için konu adına tıklayın.

## 1. [DevOps'un temelleri](konular/01-devops-temelleri.md)

Bir değişikliğin fikirden kullanıcıya ulaşması ekibinizde kaç gün sürüyor? Bu soruya verilen cevap, bir yazılım organizasyonu hakkında çoğu mimari diyagramdan daha fazla şey anlatır. DevOps, bu süreyi kısaltırken kaliteyi ve kararlılığı da artırmanın yollarını arayan bir çalışma biçimidir. Bu dokümanda DevOps'un nereden geldiğini, ne olduğunu ve ne olmadığını, onu tarif eden çerçeveleri ve nasıl ölçüldüğünü ele alıyoruz.

Başlıklar: Yazılım teslimatının evrimi · Akış verimliliği: zaman nereye gidiyor? · DevOps nedir, ne değildir? · Yaşam döngüsü: sonsuzluk modeli · Roller, ekip yapısı ve Conway yasası · CALMS: DevOps'un beş boyutu · Three Ways: üç iyileştirme yolu · Value stream mapping · Kültür: Westrum modeli ve psikolojik güvenlik · Ölçüm: DORA metrikleri · Dönüşüm tuzakları ve nereden başlamalı?

## 2. [SRE, platform engineering ve DevSecOps](konular/02-sre-platform-devsecops.md)

DevOps bir felsefe ve çalışma biçimi olarak ne yapılması gerektiğini söyler, ama bunun nasıl yapılacağını her ayrıntısıyla tarif etmez. Zamanla bu boşluğu dolduran üç uzmanlık alanı doğdu: güvenilirliği ölçülebilir bir hedefe dönüştüren **SRE**, geliştiricilerin bilişsel yükünü azaltan **platform engineering** ve güvenliği teslimat akışının içine taşıyan **DevSecOps**. Bu doküman üçünün temel kavramlarını anlatır; araçlarını ilerleyen konularda göreceğiz.

Başlıklar: SRE nedir? · SLI, SLO ve SLA · Hata bütçesi · Toil · Olay yönetimi · Platform engineering · Team Topologies · Internal Developer Platform ve golden path · DevSecOps'a giriş · Rolleri birlikte düşünmek

## 3. [Cloud computing ve AWS](konular/03-cloud-aws.md)

Bulut, DevOps'un en büyük kolaylaştırıcılarından biridir. Bir test ortamı için haftalarca sunucu beklemek yerine birkaç dakikada ortam açıp iş bitince silebilmek, altyapıyı kodla tanımlamayı ve otomatikleştirmeyi mümkün kılar. Bu dokümanda bulutun mantığını, servis ve dağıtım modellerini, AWS'nin global altyapısını ve bir DevOps mühendisinin en sık kullandığı temel servisleri ele alıyoruz.

Başlıklar: Bulut nedir? · Dağıtım ve servis modelleri · Paylaşılan sorumluluk modeli · AWS global altyapısı · AWS'ye erişim ve maliyet · IAM: kimlik ve erişim yönetimi · VPC ve ağ güvenliği · Temel servisler · Servisler birlikte: üç katmanlı mimari · Well-Architected Framework

## 4. [Git ve kaynak kod yönetimi](konular/04-git.md)

Bu dokümanlarda anlatılan hemen her şey bir Git deposunda yaşar: uygulama kodu, testler, Dockerfile'lar, pipeline tanımları, Terraform kodu, Kubernetes manifest'leri ve Helm chart'ları. Git'i sağlam öğrenmeden sonraki konuların hiçbiri yerine oturmaz. Bu dokümanda versiyon kontrolünün mantığını, Git'in iç modelini, branch ve merge stratejilerini ve GitHub ile GitLab üzerindeki iş birliği akışını ele alıyoruz.

Başlıklar: Versiyon kontrolü ve Git · Git'in üç alanı · Branch ve merge · Branching stratejileri · Değişikliği geri almak · Uzak depolarla çalışmak · GitHub, GitLab ve pull request

## 5. [Linux for DevOps](konular/05-linux.md)

DevOps dünyasında Linux her yerdedir. Sunucuların ve sanal makinelerin büyük çoğunluğu Linux çalıştırır. CI/CD runner'ları Linux üzerinde koşar ve pipeline adımlarının çoğu aslında birer shell komutudur. Kubernetes node'ları Linux çekirdeğinin özelliklerini kullanır. Bir container ise namespace ve cgroup ile izole edilmiş bir Linux process'inden ibarettir. Docker'ı, Kubernetes'i ve CI'ı anlamanın en kısa yolu Linux'u anlamaktır. Bu doküman, bir DevOps mühendisinin Linux'ta en çok ihtiyaç duyduğu kavramları anlatır.

Başlıklar: Linux mimarisi ve komutların davranışı · Dosya sistemi ve izinler · Metin işleme ve log okuma · Process'ler ve kaynaklar · systemd ve servisler · Ağ ve güvenli erişim · Bash ile otomasyon · Sorun giderme yaklaşımı

## 6. [DevOps için ağ temelleri](konular/06-ag.md)

"Uygulama çalışıyor ama erişilemiyor" vakalarının çoğu kodda değil, ağdadır. Container ayaktadır ama port yanlış adreste dinliyordur; sanal makine açıktır ama security group SSH'a izin vermiyordur; Pod çalışıyordur ama Ingress 502 döndürüyordur; yeni sürüm yayındadır ama kullanıcılar DNS önbelleği yüzünden eskisini görüyordur. Her katmanı ayrı ayrı test edebilen biri bu tür sorunları dakikalar içinde daraltır. Bu doküman, bir DevOps mühendisinin ihtiyaç duyduğu ağ kavramlarını anlatır.

Başlıklar: Ağ cihazları ve katmanlar · Bir isteğin yolculuğu · IP adresleme ve CIDR · NAT ve firewall · TCP ve UDP · DNS · HTTP ve durum kodları · TLS · Proxy ve load balancing

## 7. [Build, bağımlılık yönetimi ve test](konular/07-build.md)

Kaynak kod tek başına çalışmaz. Bağımlılıklarının indirilmesi, derlenmesi, test edilmesi ve paketlenmesi gerekir. Bu adımların toplamına **build** denir ve CI/CD pipeline'larının çekirdeğinde hep bu adımlar çalışır. Bu dokümanda build araçlarının ortak kavramlarını, bağımlılık yönetimini, örnek olarak npm ekosistemini ve test otomasyonunu ele alıyoruz.

Başlıklar: Build nedir? · Manifest, lockfile ve registry · Tedarik zinciri riskleri · npm ekosistemi · Test otomasyonu · Pipeline'ı tasarlamak

## 8. [Docker ve container](konular/08-docker.md)

"Benim makinemde çalışıyordu" cümlesi, yazılım dünyasının en eski şikâyetlerinden biridir. Geliştiricinin bilgisayarında, test sunucusunda ve üretimde farklı işletim sistemi sürümleri, farklı runtime'lar, farklı kütüphaneler ve farklı ayarlar vardır. Container'lar bu soruna cevap olarak yaygınlaştı: uygulamayı bağımlılıklarıyla birlikte, her yerde aynı çalışan bir pakete dönüştürdüler. Bu dokümanda container'ın ne olduğunu, Docker'ın nasıl çalıştığını, image'ların nasıl üretildiğini, ağ ve depolamayı, Compose'u ve üretime hazır container pratiklerini ele alıyoruz.

Başlıklar: Container ve sanal makine · Image ve container · Docker'ın mimarisi · Container izolasyonu · Container yaşam döngüsü ve operasyon · Image üretmek: Dockerfile · Registry ve image etiketleri · Ağ · Depolama · Docker Compose · Üretime hazır container'lar · Docker'dan Kubernetes'e

## 9. [Kubernetes](konular/09-kubernetes.md)

Docker bir uygulamayı her yerde aynı çalışan bir pakete dönüştürür, ama tek bir makinede. Onlarca makineye yayılmış yüzlerce container'ı hangi makinede çalıştıracağınıza, biri çöktüğünde ne olacağına, yeni sürümü kesintisiz nasıl yayınlayacağınıza ve trafiği nasıl dağıtacağınıza kim karar verecek? Kubernetes bu sorulara cevap veren bir orkestrasyon platformudur. Bu dokümanda Kubernetes'in mimarisini, temel nesnelerini, ağ ve depolama modelini, uygulama sağlığını, ölçeklemeyi, dağıtım stratejilerini, güvenliği ve Helm'i ele alıyoruz.

Başlıklar: Kubernetes nedir? · Mimari · kubectl, kubeconfig ve namespace · Pod, ReplicaSet ve Deployment · Ağ ve trafik · Yapılandırma ve depolama · Uygulama sağlığı · Kaynaklar ve ölçekleme · İleri iş yükleri · Dağıtım stratejileri · Zamanlama ve güvenlik · Helm ve Headlamp

## 10. [CI/CD süreci](konular/10-cicd.md)

CI/CD, DevOps felsefesinin en somut pratiğidir: değişikliği kullanıcıya otomatik, güvenli ve tekrarlanabilir biçimde ulaştırmak. Bu doküman araçlardan bağımsız olarak CI/CD'nin kavramlarını, uçtan uca bir pipeline'ın aşamalarını ve pipeline tasarımını anlatır. Sonraki konularda aynı kavramları Jenkins, GitHub Actions ve GitLab CI ile uygulanmış hâliyle göreceğiz.

Başlıklar: CI/CD'den önce · Continuous integration, delivery ve deployment · Uçtan uca pipeline · Pipeline tasarımı · Push tabanlı CD ve GitOps · Güvenlik ve ölçüm

## 11. [Jenkins ile CI/CD](konular/11-jenkins.md)

Jenkins, CI/CD dünyasının en eski ve hâlâ en yaygın araçlarından biridir. Birçok kurumun pipeline'ı onun üzerinde çalışır. Jenkins'in gücü esnekliğidir: neredeyse her şeyi yapabilir ve neredeyse her araçla konuşabilir. Aynı esneklik, bakımın tamamen kuruma ait olması anlamına gelir. Bu dokümanda Jenkins'in mimarisini, pipeline as code yaklaşımını, kaynak kod yönetimiyle entegrasyonunu, agent'ları ve güvenli işletimini ele alıyoruz.

Başlıklar: Jenkins nedir? · Mimari: controller ve agent'lar · Pipeline as code · Kaynak kod yönetimi ve multibranch · Agent'lar, Docker ve Kubernetes · Güvenlik ve işletim

## 12. [GitHub Actions ve GitLab CI](konular/12-github-gitlab.md)

Jenkins ayrı bir sunucudur. GitHub Actions ve GitLab CI ise kodun durduğu platformun içindedir: depo, pull ya da merge request ve pipeline aynı yerde yaşar. Bu dokümanda iki aracın kavramlarını, birbirine nasıl eşlendiğini ve güvenli kullanımını ele alıyoruz.

Başlıklar: GitHub Actions'ın temelleri · Secret'lar, değişkenler ve kimlik · Job'lar, runner'lar ve yürütme · GitHub Actions ile CI/CD · GitLab CI/CD · Sık yapılan hatalar

## 13. [Registry ve güvenli teslimat](konular/13-registry.md)

Jenkins, GitHub Actions ve GitLab CI ile kurulan pipeline'ların ortak varış noktası bir container registry'dir. Image'ın nerede durduğu, kimin erişebildiği, nasıl doğrulandığı ve hedef ortama nasıl ulaştığı, teslimat güvenliğinin merkezindedir. Bu dokümanda registry kavramlarını, kurum içi bir registry olan Harbor'ı ve pipeline'lardan Docker ve Kubernetes'e güvenli teslimatı ele alıyoruz.

Başlıklar: Image adresinin anatomisi · Registry seçenekleri · Harbor · CI'dan registry'ye · Docker ve Kubernetes'e dağıtım · Sorun giderme ve sık hatalar

## 14. [DevSecOps](konular/14-devsecops.md)

DevSecOps, güvenliği teslimatın sonundaki bir manuel onaydan çıkarıp pipeline'ın doğal bir parçası hâline getirmektir. SRE, platform ve DevSecOps konusunda kavramına giriş yaptık; bu dokümanda tehdit modellemeyi, OWASP risklerini, statik analizi, bağımlılık ve image taramasını, secret yönetimini, yazılım tedarik zinciri güvenliğini ve politika olarak kodu araçlarıyla birlikte ele alıyoruz.

Başlıklar: Temeller · OWASP Top 10 · Secret yönetimi · Statik analiz ve SonarQube · Bağımlılık ve image taraması · Yazılım tedarik zinciri · Uçtan uca güvenli teslimat · Politika olarak kod

## 15. [Altyapı kodu: Terraform ve Ansible](konular/15-iac-terraform.md)

Altyapı değişikliği kişilere ve arayüzlere bağlı kaldığında tekrar üretmek, incelemek ve güvenle sürdürmek zorlaşır. Aynı ortamı iki kişi farklı sırayla ve farklı ayrıntılarla kurar; kimin neyi neden değiştirdiği bulunamaz; test ve üretim zamanla birbirinden ayrışır. **Infrastructure as Code** (IaC), altyapıyı makine tarafından işlenebilir tanımlarla yönetmek ve bu tanımları versiyon kontrolünde tutmaktır. Bu dokümanda IaC yaklaşımlarını, Ansible'ın yerini ve Terraform'u ayrıntılı olarak ele alıyoruz.

Başlıklar: IaC yaklaşımları · Terraform ve Ansible · Terraform'un mimarisi · HCL'nin yapı taşları · State · Yaşam döngüsü ve güvenli iş akışı · Tekrar kullanım: değişkenler, modüller · Terraform uygulamada

## 16. [GitOps ve Argo CD](konular/16-gitops.md)

CI/CD konusunde dağıtımın iki modelini gördük: pipeline'ın değişikliği cluster'a ittiği push modeli ve cluster içindeki bir ajanın Git'i izleyip kendini eşitlediği pull modeli. **GitOps**, Git'i operasyonun tek doğruluk kaynağı yapan ve değişiklikleri inceleme, denetim ve otomatik uzlaştırma ile yöneten bu ikinci modelin adıdır. Bu dokümanda GitOps'un prensiplerini, istenen durum ile gerçek durumun uzlaştırılmasını, ortamlar arası terfiyi ve Argo CD'yi ele alıyoruz.

Başlıklar: GitOps nedir? · İstenen durum ve uzlaştırma · Ortamlar ve terfi · Geri alma · Argo CD · Yeni bir image cluster'a nasıl gelir? · Progressive delivery: Argo Rollouts

## 17. [Observability ve Prometheus](konular/17-observability.md)

DevOps sonsuzluk döngüsünün son halkası izlemedir: kod yazıldı, paketlendi ve dağıtıldı; şimdi canlıda ne olduğunu görmek gerekir. Bu dokümanda monitoring ile observability farkını, telemetri sinyallerini, neyin ve nasıl ölçüleceğini, OpenTelemetry'yi, Prometheus'u, alarmları, Grafana'yı ve Kubernetes'te izlemeyi ele alıyoruz.

Başlıklar: Monitoring ve observability · Telemetri sinyalleri · Neyi ölçmeli? · OpenTelemetry · Prometheus · Alarmlar ve Grafana · Kubernetes'te izleme

## 18. [ELK ile log yönetimi](konular/18-elk.md)

Önceki konuda metrikleri Prometheus ile topladık. Bu doküman üç sinyalin ikincisine, loglara odaklanıyor: dağınık log dosyalarından aranabilir, ilişkilendirilebilir ve alarm üreten bir log platformuna geçmek. Elastic Stack'in bileşenlerini, logların toplanması ve işlenmesini, Kibana ile aramayı, saklama ve güvenliği ve alternatif log platformlarını ele alıyoruz.

Başlıklar: Neden merkezi log? · Elastic Stack · Toplama ve işleme · Kibana ile arama · İşletme

## 19. [Yapay zekâ ve DevOps](konular/19-yapay-zeka.md)

Yapay zekâ araçları DevOps mühendisinin günlük işine hızla girdi: Dockerfile taslağı yazmak, kırmızı bir pipeline'ın logunu okumak, Terraform planını özetlemek, bir alarm kuralı üretmek. Bu doküman araç tanıtımı değil, bir çalışma modelidir. Büyük dil modellerinin nasıl çalıştığını ve neden yanıldığını, yapay zekâ çıktısının DevOps araçlarıyla nasıl doğrulandığını, ajanları ve araç bağlantılarını, operasyonda yapay zekâyı ve güvenlik ile sorumluluk sınırlarını ele alıyoruz.

Başlıklar: Temel kavramlar · Modeller ve araçlar · DevOps'ta kullanım alanları · Ajanlar ve MCP · AIOps ve bulutta yapay zekâ · Güvenlik ve sorumluluk

## [Sözlük](konular/sozluk.md)

Dokümanlarda geçen terimlerin kısa açıklamaları.
