# DevOps Practitioner dokümanları

DevOps'un kültür, süreç ve araç tarafını kapsayan konu anlatımları, konu özeti, sunum ve faydalı bağlantılar.

## Başlarken

- [Konu özeti](OZET.md): bütün konuların kısa özeti ve başlıkları
- [DevOps Foundation sunumu (PDF)](docs/DevOps_Foundation_Sunum.pdf): kavramlar, pratikler ve araçlar, konu sırasıyla
- [Faydalı bağlantılar](FAYDALI_BAGLANTILAR.md): resmî dokümantasyon, DORA, SRE ve ücretsiz pratik ortamları
- [Sözlük](konular/sozluk.md)

## Konular

| # | Konu | Kısaca |
|---|---|---|
| 1 | [DevOps'un temelleri](konular/01-devops-temelleri.md) | Bir değişikliğin fikirden kullanıcıya ulaşması ekibinizde kaç gün sürüyor? |
| 2 | [SRE, platform engineering ve DevSecOps](konular/02-sre-platform-devsecops.md) | DevOps bir felsefe ve çalışma biçimi olarak ne yapılması gerektiğini söyler, ama bunun nasıl yapılacağını her ayrıntısıyla tarif etmez. |
| 3 | [Cloud computing ve AWS](konular/03-cloud-aws.md) | Bulut, DevOps'un en büyük kolaylaştırıcılarından biridir. |
| 4 | [Git ve kaynak kod yönetimi](konular/04-git.md) | Bu dokümanlarda anlatılan hemen her şey bir Git deposunda yaşar: uygulama kodu, testler, Dockerfile'lar, pipeline tanımları, Terraform kodu, Kubernetes manifest'leri ve Helm chart'ları. |
| 5 | [Linux for DevOps](konular/05-linux.md) | DevOps dünyasında Linux her yerdedir. |
| 6 | [DevOps için ağ temelleri](konular/06-ag.md) | "Uygulama çalışıyor ama erişilemiyor" vakalarının çoğu kodda değil, ağdadır. |
| 7 | [Build, bağımlılık yönetimi ve test](konular/07-build.md) | Kaynak kod tek başına çalışmaz. |
| 8 | [Docker ve container](konular/08-docker.md) | "Benim makinemde çalışıyordu" cümlesi, yazılım dünyasının en eski şikâyetlerinden biridir. |
| 9 | [Kubernetes](konular/09-kubernetes.md) | Docker bir uygulamayı her yerde aynı çalışan bir pakete dönüştürür, ama tek bir makinede. |
| 10 | [CI/CD süreci](konular/10-cicd.md) | CI/CD, DevOps felsefesinin en somut pratiğidir: değişikliği kullanıcıya otomatik, güvenli ve tekrarlanabilir biçimde ulaştırmak. |
| 11 | [Jenkins ile CI/CD](konular/11-jenkins.md) | Jenkins, CI/CD dünyasının en eski ve hâlâ en yaygın araçlarından biridir. |
| 12 | [GitHub Actions ve GitLab CI](konular/12-github-gitlab.md) | Jenkins ayrı bir sunucudur. |
| 13 | [Registry ve güvenli teslimat](konular/13-registry.md) | Jenkins, GitHub Actions ve GitLab CI ile kurulan pipeline'ların ortak varış noktası bir container registry'dir. |
| 14 | [DevSecOps](konular/14-devsecops.md) | DevSecOps, güvenliği teslimatın sonundaki bir manuel onaydan çıkarıp pipeline'ın doğal bir parçası hâline getirmektir. |
| 15 | [Altyapı kodu: Terraform ve Ansible](konular/15-iac-terraform.md) | Altyapı değişikliği kişilere ve arayüzlere bağlı kaldığında tekrar üretmek, incelemek ve güvenle sürdürmek zorlaşır. |
| 16 | [GitOps ve Argo CD](konular/16-gitops.md) | CI/CD konusunde dağıtımın iki modelini gördük: pipeline'ın değişikliği cluster'a ittiği push modeli ve cluster içindeki bir ajanın Git'i izleyip kendini eşitlediği pull modeli. |
| 17 | [Observability ve Prometheus](konular/17-observability.md) | DevOps sonsuzluk döngüsünün son halkası izlemedir: kod yazıldı, paketlendi ve dağıtıldı; şimdi canlıda ne olduğunu görmek gerekir. |
| 18 | [ELK ile log yönetimi](konular/18-elk.md) | Önceki konuda metrikleri Prometheus ile topladık. |
| 19 | [Yapay zekâ ve DevOps](konular/19-yapay-zeka.md) | Yapay zekâ araçları DevOps mühendisinin günlük işine hızla girdi: Dockerfile taslağı yazmak, kırmızı bir pipeline'ın logunu okumak, Terraform planını özetlemek, bir alarm kuralı üretmek. |

## Ek kaynaklar (PDF)

İnternette paylaşılan, konulara göre seçilmiş notlar ve rehberler `docs/` klasöründe:

- [1. Introduction to DevSecOps](docs/1.%20Introduction%20to%20DevSecOps.pdf)
- [1. Kubernetes Fundamentals](docs/1.%20Kubernetes%20Fundamentals.pdf)
- [50 Monitoring Mistakes](docs/50%20Monitoring%20Mistakes.pdf)
- [CI-CD — Simple Notes](docs/CI-CD%20—%20Simple%20Notes.pdf)
- [CI_CD_Temelleri_DevOps_Practitioner_TR](docs/CI_CD_Temelleri_DevOps_Practitioner_TR.pdf)
- [DEVOPS ENGINEERING](docs/DEVOPS%20ENGINEERING.pdf)
- [DOCKER COMPLETE NOTES](docs/DOCKER%20COMPLETE%20NOTES.pdf)
- [DOCKER Complete Learning Notes](docs/DOCKER%20Complete%20Learning%20Notes.pdf)
- [DevOps Ek ve Basitleştirilmiş Slaytlar](docs/DevOps%20Ek%20ve%20Basitleştirilmiş%20Slaytlar.pdf)
- [DevOps-Essential-2](docs/DevOps-Essential-2.pdf)
- [GitLab CI-CD Complete Guide 2026](docs/GitLab%20CI-CD%20Complete%20Guide%202026.pdf)
- [KUBERNETES Complete Commands Reference Guide](docs/KUBERNETES%20Complete%20Commands%20Reference%20Guide.pdf)
- [Kubernetes — 100 Scenarios, Explained](docs/Kubernetes%20—%20100%20Scenarios,%20Explained.pdf)
- [LINUX FOR DEVOPS](docs/LINUX%20FOR%20DEVOPS.pdf)
- [Learn GIT Like never before](docs/Learn%20GIT%20Like%20never%20before.pdf)
- [MODERN DEVOPS 3 CORE PRACTICES](docs/MODERN%20DEVOPS%203%20CORE%20PRACTICES.pdf)
- [Platform Engineering Notes](docs/Platform%20Engineering%20Notes.pdf)
- [TERRAFORM COMMANDS CHEATSHEET](docs/TERRAFORM%20COMMANDS%20CHEATSHEET.pdf)
- [Top 500 Troubleshoot Like a Pro Kubernetes Troubleshooting Commands](docs/Top%20500%20Troubleshoot%20Like%20a%20Pro%20Kubernetes%20Troubleshooting%20Commands.pdf)
- [jenkins-101](docs/jenkins-101.pdf)
- [networking for devops](docs/networking%20for%20devops.pdf)
- [ÖZET](docs/—------ÖZET—-------.pdf)
