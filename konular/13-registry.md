# Registry ve güvenli teslimat

Jenkins, GitHub Actions ve GitLab CI ile kurulan pipeline'ların ortak varış noktası bir container registry'dir. Image'ın nerede durduğu, kimin erişebildiği, nasıl doğrulandığı ve hedef ortama nasıl ulaştığı, teslimat güvenliğinin merkezindedir. Bu dokümanda registry kavramlarını, kurum içi bir registry olan Harbor'ı ve pipeline'lardan Docker ve Kubernetes'e güvenli teslimatı ele alıyoruz.

## Image adresinin anatomisi

Bir image adresinin her parçası bir soruya cevap verir.

Registry adı "nerede?", proje "kimin?", repository "ne?", etiket "hangi sürüm?" sorusunu cevaplar. Etiket değişebilir; aynı etiket sonradan başka bir image'ı gösterebilir. İçerik özeti (digest) ise "tam olarak hangisi?" sorusunun değişmez cevabıdır. Kısa yazımlar Docker tarafından tamamlanır: yalnızca "nginx" yazıldığında Docker Hub'daki resmi image'ın latest etiketi anlaşılır. Registry yazılmadığında Docker Hub'ın, etiket yazılmadığında latest'in varsayılması, üretimde istenmeyen sürprizlere yol açar.

## Registry seçenekleri

Docker Hub açık kaynak dağıtımı için varsayılan registry'dir ve resmi image'ların evidir. GitHub Container Registry kodun GitHub'da olduğu durumlarda doğal bir seçimdir. Amazon ECR gibi bulut registry'leri IAM ile erişim sağlar ve iş yükü o buluttaysa tercih edilir. **Harbor** ise kurumun kendi sunucusuna kurduğu, CNCF'in mezun projelerinden biri olan bir registry'dir: LDAP ve OIDC ile kimlik, CI/CD için robot hesaplar, entegre zafiyet taraması, proxy önbellek ve registry'ler arası çoğaltma sunar. Yaygın bir yaklaşım, geliştirme için Docker Hub, kurum içi teslimat için Harbor kullanmaktır.

Docker Hub anonim ve ücretsiz kullanıcılara saatlik indirme sınırları uygular; ücretli planlarda bu sınır kalkar. CI'da anonim indirme yapmak, özellikle ortak bir ağ adresi arkasındaki runner'larda, pipeline'ları "too many requests" hatasıyla kırmanın en kolay yoludur. Çözüm, CI'da her zaman giriş yapmak, giriş için parola yerine erişim token'ı kullanmak ve kurum içinde bir proxy önbellek kullanmaktır.

Registry konusunun temel ilkesi, CI/CD konusunden beri tekrarladığımız "bir kez üret, aynı image'ı ilerlet" ilkesidir. Aynı commit farklı bir base image ile yeniden derlendiğinde farklı bir image üretebilir; bu yüzden staging'de doğrulanan image'ın digest'i üretime taşınır.

## Harbor

Harbor'ın altı temel yeteneği vardır. Image'lar **projelerde** durur ve erişim rollerle verilir. **Robot hesaplar** CI/CD için süreli ve kapsamlı makine kimlikleri sunar. **Zafiyet taraması** image gönderildiği anda Trivy ile yapılır ve politikalarla zafiyetli image'ların kullanımı engellenebilir. **İmza ve yazılım envanteri** desteklenir. **Proxy önbellek**, Docker Hub ve diğer registry'lerin içeriğini kurum içinde önbelleğe alır. **Çoğaltma** ise Harbor kurulumları ve registry'ler arasında senkronizasyon sağlar.

Harbor bir dizi servisten oluşur.

Önde bir nginx proxy bulunur. Core servisi API'yi, kimlik doğrulamayı ve politikaları, portal web arayüzünü, registry servisi image katmanlarını yönetir. Jobservice çoğaltma, tarama ve çöp toplama işlerini yürütür; trivy-adapter zafiyet tarayıcısıdır. Arkada metadata için PostgreSQL, önbellek ve kuyruk için Redis ve disk ya da S3 uyumlu bir depolama bulunur. Durum tutan parçalar, yani PostgreSQL, Redis ve image depolaması, yedeklenmesi gerekenlerdir. Yüksek erişilebilirlik gerekirse Harbor, harici bir veritabanı ve nesne depolama ile Kubernetes'e kurulur.

Harbor'da erişim proje bazında verilir. Proje yöneticisi her şeyi yapabilir; maintainer image gönderip çekebilir ve etiket silebilir; geliştirici image gönderip çekebilir; misafir yalnızca okuyabilir; sınırlı misafir yalnızca image çekebilir. Herkese açık projelerden herkes image çekebilir ve base image'lar için uygundur; özel projeler ise varsayılan tercihtir. Kişileri tek tek eklemek yerine dizin servisi gruplarını projelere bağlamak yönetimi kolaylaştırır.

CI pipeline'larına kişisel bir parola girmez; **robot hesap** kullanılır. İyi bir pratik, image gönderen ve çeken kimlikleri ayırmaktır: CI'ın kullandığı bir robot image gönderip çekebilir, Kubernetes'in kullandığı robot yalnızca image çekebilir. Böylece cluster ele geçirilse bile saldırgan registry'ye zararlı bir image gönderemez. Robot hesaplara son kullanma tarihi konmalı ve secret'ları CI'ın secret deposunda tutulmalıdır. Robot adlarında bulunan özel karakter nedeniyle shell'de tek tırnak kullanmak gerekir.

İki politika registry'nin düzenini korur. **Tag immutability**, sürüm etiketlerinin üzerine yazılmasını engeller; aynı etikete ikinci kez image göndermek reddedilir. Bu kural sürüm etiketlerine uygulanmalı, latest gibi kayan etiketlere uygulanmamalıdır. **Retention** ise eski image'ların kontrollü silinmesini sağlar; örneğin son yirmi image ve son otuz günde çekilenler tutulur, gerisi silinir. Silinen image'ların disk alanı ancak **çöp toplama** çalıştığında geri kazanılır; çöp toplama planlanmazsa disk yine dolar.

## CI'dan registry'ye

Araç değişse de commit'ten registry'ye giden akış ve ürettiği kanıtlar aynıdır.

Image, sürüm numarası ve kısa commit kimliğinden oluşan izlenebilir bir etiketle derlenir. Trivy ile taranır ve yüksek ve kritik açıklar için bir eşik aşılırsa pipeline durur. Image registry'ye gönderilir ve registry bir digest döndürür. İmzalanır ve digest kaydedilir; bu digest dağıtım adımının girdisidir. Her adım bir kanıt bırakır: build logu, tarama raporu, digest. Bir denetimde "üretimdeki image hangi koddan geldi ve hangi taramadan geçti?" sorusu bu kanıtlarla cevaplanır.

GitHub Actions'ta Harbor'a giriş robot hesapla yapılır; kullanıcı adı bir değişken, parola bir secret olarak tutulur ve build eyleminin çıktısındaki digest sonraki job'a taşınır. Harbor iç ağdaysa platformun runner'ları ona erişemez ve kurum içi runner gerekir. GitLab ya da herhangi bir runner'da parolayı komut satırına yazmadan standart girdiden vererek giriş yapılır ve image tek komutla derlenip gönderilir. Jenkins'te ise robot hesabın bilgileri credential deposunda tutulur ve tek tırnak kuralıyla kullanılır; Kubernetes üzerinde çalışan agent'larda Docker soketi yerine Buildah ya da BuildKit gibi araçlar tercih edilir.

## Docker ve Kubernetes'e dağıtım

Bir Docker sunucusuna dağıtım, hedef sunucuda image'ı tam digest ile çekip servisi yeniden build etmeden başlatmaktır. Bu komutların hedef sunucuda çalışması gerekir; CI job'ının içindeki geçici Docker'a dağıtım yapılırsa job bitince her şey kaybolur ve uygulama sunucusu hiç güncellenmemiş olur.

Kubernetes'in özel bir registry'den image çekebilmesi için bir **imagePullSecret** gerekir. Bu secret, yalnızca image çekme yetkisi olan bir robot hesapla oluşturulur ve her Pod'a tek tek yazmak yerine namespace'in ServiceAccount'una bağlanır; böylece o namespace'teki bütün Pod'lar onu otomatik kullanır. Secret namespace'e ait olduğu için her namespace'te ayrı oluşturulur ve Git'te düz metin olarak değil, Sealed Secrets ya da External Secrets gibi yöntemlerle yönetilmelidir.

Başarılı bir dağıtımın kanıtı üç sorudur: rollout tamamlandı mı, bir HTTP smoke testi geçti mi ve beklenen sürüm gerçekten çalışıyor mu? Rollout'un tamamlanması altyapı açısından önemlidir ama yeterli değildir; Pod'lar ayakta olup uygulama yanlış cevap veriyor olabilir. Geri dönüş için ise iki şey gerekir: önceki digest ve geriye uyumlu veri değişikliği.

Şimdiye kadar anlatılan parçaların hepsi tek bir teslimat hattında birleşir.

Commit, test, build, Trivy taraması, Harbor'a gönderim, imza, digest kaydı, staging dağıtımı, smoke ve uçtan uca testler, onay, üretim ve izleme. Bu hatta pipeline'ın kendisi dağıtım yapar, yani push modeli kullanılır; GitOps konusunde cluster'ın Git'i izleyip kendini eşitlediği pull modeline geçeceğiz.

## Sorun giderme ve sık hatalar

Registry hataları genellikle mesajlarından tanınır. "Certificate signed by unknown authority" hatası, istemcinin registry sertifikasını imzalayan otoriteyi tanımadığını gösterir; çözüm o otoritenin sertifikasını Docker'ın ya da containerd'nin sertifika dizinine koymaktır, TLS'i kapatmak değil. "Authentication required" girişin yapılmadığını ya da robot adının yanlış yazıldığını, "requested access to the resource is denied" robotun gönderme izni olmadığını, "too many requests" Docker Hub sınırına takıldığını, "manifest unknown" ilgili etiketin olmadığını ya da retention tarafından silindiğini, "configured as immutable" ise aynı sürüm etiketine tekrar gönderim yapıldığını gösterir. Kubernetes'teki ImagePullBackOff hatası secret, ağ, sertifika ya da politika kaynaklı olabilir ve Pod olaylarından teşhis edilir.

Registry ve teslimatta sık yapılan hatalar şunlardır: CI'da yönetici hesabı kullanmak, gönderme ve çekme için aynı kimliği kullanmak, latest etiketiyle dağıtım yapmak, TLS'i kapatmak için güvensiz registry ayarı kullanmak, Docker Hub'dan anonim indirme yapmak, taranmamış image'ları üretime çıkarmak, çöp toplamasız retention ve yedeksiz Harbor. Registry teslimat hattının kasasıdır: içine kimin ne koyduğu, kimin ne aldığı ve içinde ne olduğu her zaman kanıtlanabilir olmalıdır.

> Etiket değişebilir, digest değişmez; dağıtım digest ile yapılır. Harbor projeler, robot hesaplar, tarama, değiştirilemez etiketler ve proxy önbellek ile kurum içi teslimatı güvenli kılar. Kubernetes yalnızca çekme yetkili bir robotla, ServiceAccount'a bağlanmış bir imagePullSecret üzerinden image çeker.
