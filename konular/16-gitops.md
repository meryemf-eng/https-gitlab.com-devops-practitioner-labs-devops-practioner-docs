# GitOps ve Argo CD

CI/CD konusunde dağıtımın iki modelini gördük: pipeline'ın değişikliği cluster'a ittiği push modeli ve cluster içindeki bir ajanın Git'i izleyip kendini eşitlediği pull modeli. **GitOps**, Git'i operasyonun tek doğruluk kaynağı yapan ve değişiklikleri inceleme, denetim ve otomatik uzlaştırma ile yöneten bu ikinci modelin adıdır. Bu dokümanda GitOps'un prensiplerini, istenen durum ile gerçek durumun uzlaştırılmasını, ortamlar arası terfiyi ve Argo CD'yi ele alıyoruz.

## GitOps nedir?

GitOps, Git'te tanımlanan istenen durum ile gerçek sistem durumunu sürekli uzlaştıran bir operasyon modelidir.

Üç parçası vardır. **Git doğruluk kaynağıdır**: uygulama ve altyapının istenen durumu Git'te declarative olarak tutulur ve değişiklikler commit ve pull request ile izlenir. **Reconciler operatördür**: cluster içindeki bir ajan belirli aralıklarla Git'i okur, gerçek durumla karşılaştırır ve fark varsa düzeltir. **Cluster gerçek durumdur**: bir Kubernetes cluster'ı, sanal makineler ya da bulut kaynakları olabilir. Bu fikir aslında Kubernetes'in kendisinde de vardır: Kubernetes istenen durumu sürekli uzlaştıran bir sistemdir; GitOps aynı fikri bir seviye yukarıya, Git'e taşır.

GitOps CI/CD'yi ortadan kaldırmaz; sorumlulukları netleştirir.

CI testleri çalıştırır, image üretir, güvenlik taramalarını yapar ve değişmez bir digest yayınlar; hedefi dağıtılabilir bir artifact üretmektir. Ortam deposunda image referansı ya da Helm değerleri güncellenir, değişiklik pull request ile incelenir ve birleştirildiğinde istenen durum değişir; bu depo dağıtım niyetini taşır. GitOps ajanı ise Git'i izler, farkı bulur, değişikliği cluster'a uygular ve drift varsa düzeltir. Dağıtım erişimi cluster içindeki ajana bırakılır; pipeline cluster'a dokunmaz.

GitOps'u araçtan bağımsız tanımlayan dört prensip vardır. Sistem istenen durum olarak **declarative** biçimde tanımlanır. Durum versiyon kontrolünde **sürümlenir** ve geçmişi değiştirilemez biçimde izlenir. Ajanlar istenen durumu kaynaktan **otomatik olarak çeker**. Ajan gerçek durumu istenen durumla **sürekli uzlaştırır**. Bu dördü birlikte yoksa sistem Git ile dağıtım yapıyor olabilir ama tam bir GitOps modeli değildir; örneğin Git'e gönderim yapıldığında kubectl çalıştıran bir pipeline, çekme ve sürekli uzlaştırma olmadığı için GitOps sayılmaz.

## İstenen durum ve uzlaştırma

Reconciler iki dünyayı sürekli karşılaştırır. Git'te üç kopya, belirli bir digest ile bir image ve tanımlı kaynak sınırları yazıyor olabilir; cluster'da ise bir kopya, eski bir image ve elle uygulanmış bir yama bulunabilir. Reconciler farkı görür ve gerekli değişiklikleri yaparak istenen duruma yakınsar.

**Drift**, cluster'daki gerçek durumun Git'teki durumdan sapmasıdır. Beş kaynağı vardır: elle yapılan değişiklikler, bir operatör ya da otomatik ölçekleyici gibi dış aktörler, başka bir pipeline ya da script ile yapılan bant dışı dağıtımlar, yanlış branch ya da yol gibi yapılandırma uyumsuzlukları ve yarım kalmış uygulamalar. GitOps'ta drift gizli bir durum değildir; reconciler farkı görünür kılar ve politikaya göre ya otomatik düzeltir ya da operatöre "senkron dışı" sinyali verir.

**Self-healing**, bu sapmanın otomatik olarak geri alınmasıdır: birisi kopya sayısını üçten bire düşürdüğünde reconciler farkı tespit eder, yamayı uygular ve sistem yeniden üç kopyaya döner. Self-heal bir politika kararıdır: bazı ekipler üretimde otomatik düzeltme ister, bazıları kontrollü ve manuel senkronizasyonu tercih eder. Acil bir durumda cluster'da elle bir değişiklik yapmak gerekirse self-heal'in onu kısa sürede geri alacağı bilinmelidir.

GitOps'ta bir üretim değişikliğinin yolu altı adımdır: değişiklik yapılır, pull request açılıp diff incelenir, lint ve güvenlik kontrolleri çalışır, değişiklik birleştirilir ve istenen durum güncellenir, ajan cluster'ı senkronize eder ve sağlık ile senkron durumu doğrulanır. Üretime giden yolda kimse kubectl çalıştırmaz; her değişiklik bir pull request'tir ve kim, ne zaman, neden değiştirdi sorusunun cevabı Git geçmişindedir.

## Ortamlar ve terfi

**Promotion**, yani terfi, aynı artifact'i ortamlar arasında kontrollü olarak ilerletmektir.

Geliştirme ortamında hızlı geri bildirim, staging'de smoke ve entegrasyon testleri, üretimde onay ve kademeli dağıtım yapılır; üç ortamda da aynı digest çalışır. Terfi yeni bir image derlemek değil, test edilmiş aynı artifact referansını bir sonraki ortamın istenen durumuna taşımaktır; pratikte ortam deposundaki bir dosyada digest'i değiştiren bir pull request'tir.

GitOps'ta "Git'te ne yazıyorsa o çalışır" diyebilmek için image referansının değişmez olması gerekir. latest gibi bir etiket kötü bir seçimdir: aynı Git commit'i farklı zamanlarda farklı image'lar çekebilir ve Git'teki metin aynı kalırken çalışan kod değişebilir. Bir sürüm etiketi, registry'de değiştirilemez olarak korunuyorsa yönetilebilir bir seçimdir. Digest ise en kesin seçimdir; içeriğin doğrudan kimliğidir.

GitOps'ta ortam deposuna yazabilen, üretimi değiştirebilir. Bu yüzden GitOps güvenliği büyük ölçüde depo güvenliğidir: pull request kuralları, kod sahipliği, secret yönetimi ve erişim kontrolü birlikte düşünülmelidir. Secret'lar Git'e düz metin olarak konmaz; Sealed Secrets ya da SOPS ile şifrelenmiş hâlde konur ya da External Secrets ile bir kasadan eşitlenir.

## Geri alma

GitOps'ta geri alma, cluster'a elle eski bir manifest uygulamak değildir. Üç yol vardır. **Git revert** ile hatalı commit tersine çevrilir ve yeni bir commit olarak birleştirilir; denetim izi korunur ve en genel yöntem budur. **Sürüm sabitleme** ile image ya da chart referansı önceki çalışan değere alınır; terfi depolarında hızlı ve okunaklıdır. **Araç geçmişi** ile Argo CD ya da Helm'in revision geçmişinden önceki duruma dönülebilir, ama kalıcı istenen durumun yine Git ile uyumlu hâle getirilmesi gerekir; aksi hâlde bir sonraki senkronizasyon sistemi tekrar hatalı sürüme götürür. Kalıcı geri almanın kaynağı her zaman Git'tir.

## Argo CD

**Argo CD**, Kubernetes için declarative, pull tabanlı bir sürekli teslimat aracıdır ve CNCF'in mezun projelerinden biridir.

Bileşenleri şunlardır. **repo-server** kaynağı Git'ten, Helm deposundan ya da OCI registry'den çeker ve Helm ya da Kustomize ile manifest'leri üretir. **application-controller** istenen durum ile canlı durumu karşılaştırır, senkronizasyonu ve sağlık değerlendirmesini yapar. **api-server** web arayüzünü, komut satırı aracını ve API'yi sunar; kimlik ve erişim kontrolü buradadır. **applicationset-controller** bir şablondan çok sayıda uygulama üretir. Bir önbellek servisi ve isteğe bağlı olarak tek oturum açma ve bildirim bileşenleri de bulunur. Argo CD cluster içinde çalışır ve Git'i izler; bu dokümanın başından beri anlattığımız reconciler'ın somut hâlidir.

Argo CD'nin temel nesnesi **Application**'dır ve üç parçası vardır. **source** Git deposunu, branch ya da tag'i ve yolu belirtir; bir Helm chart ya da OCI kaynağı da olabilir. **destination** hangi cluster ve hangi namespace'e dağıtım yapılacağını belirtir. **syncPolicy** ise otomatik senkronizasyonun açık olup olmadığını belirler; açıksa Git değiştiğinde değişiklik otomatik uygulanır. **prune** Git'ten silinen kaynakların cluster'dan da silinmesini, **selfHeal** elle yapılan sapmaların geri alınmasını sağlar. Application'ın kendisi de Git'te tutulmalıdır; uygulamaları arayüzden oluşturmak öğrenme için kolaydır ama GitOps ilkesine aykırıdır. Üretimde "app of apps" deseniyle uygulama tanımları da Git'ten yönetilir.

Argo CD iki ayrı soru sorar. Birincisi "Git ile aynı mı?" sorusudur: canlı durum Git ile aynıysa **Synced**, fark varsa **OutOfSync**, karşılaştırma yapılamıyorsa **Unknown**. İkincisi "uygulama sağlıklı mı?" sorusudur: kaynaklar hazırsa **Healthy**, dağıtım sürüyorsa **Progressing**, çöken Pod'lar ya da başarısız işler varsa **Degraded**. Bir uygulama Synced ama Degraded olabilir: cluster tam olarak Git'teki gibidir ama uygulama bozuktur; sorun Git'teki içeriktedir. Hedef Synced ve Healthy'nin birlikte sağlanmasıdır. Senkronizasyon seçenekleri arasında, otomatik ölçekleyicinin değiştirdiği kopya sayısı gibi başka bir controller'ın yönettiği alanları yok saymak da bulunur; aksi hâlde Argo CD o alanı sürekli Git'teki değere geri çekmeye çalışır.

## Yeni bir image cluster'a nasıl gelir?

GitOps'ta yeni bir image cluster'a şöyle gelir.

CI image'ı derler, registry'ye gönderir ve digest'i alır. Ortam deposunda ilgili değer dosyasındaki digest'i günceller ve bir commit atar ya da pull request açar. Değişiklik incelenip birleştirilir. Argo CD değişikliği görür ve cluster'ı senkronize eder. Yaygın bir pratik, geliştirme ortamına doğrudan commit, staging ve üretime ise pull request ile terfidir. Argo CD Image Updater gibi araçlar registry'yi izleyip bu commit'i kendileri de atabilir. CI'ın token'ı yalnızca ortam deposuna yazma yetkisi taşır; cluster kimliği yoktur. Push modelinden fark şudur: CI'ın cluster erişimi yoktur ve her değişiklik Git'te iz bırakır. Önceki konularda yazılan pipeline'ların "dağıtım" adımı, GitOps'ta bu commit adımıyla yer değiştirir.

## Progressive delivery: Argo Rollouts

Kubernetes'in Deployment nesnesi canary ve blue-green stratejilerini tek başına uygulayamaz. **Argo Rollouts** bu boşluğu doldurur: Deployment'ın yerine bir Rollout nesnesi kullanılır ve stratejinin adımları tanımlanır; örneğin trafiğin yüzde yirmisini yeni sürüme göndermek, beş dakika beklemek, yüzde ellisine çıkmak ve manuel onay beklemek. En güçlü özelliği analiz şablonlarıdır: Prometheus'taki hata oranı gibi bir metrik belirlenen eşiği aşarsa dağıtım otomatik olarak geri alınır. Rollout tanımı da Git'te durur ve Argo CD onun sağlık durumunu anlar. GitOps neyin dağıtılacağını, Rollouts nasıl dağıtılacağını yönetir.

Argo CD'nin en büyük alternatifi, yine CNCF'in mezun projelerinden biri olan **Flux**'tır; daha az arayüz, daha çok Kubernetes nesnesi odaklı bir yaklaşımı vardır. Kavramlar ikisinde de aynıdır.

> GitOps'ta Git istenen durumun kaynağıdır ve cluster içindeki ajan gerçek durumu sürekli onunla uzlaştırır. Pull modeli dağıtım kimliğini cluster içinde tutar, drift'i görünür kılar ve self-heal ile düzeltir. Terfi aynı değişmez artifact'i ilerletmektir; kalıcı geri almanın kaynağı her zaman Git'tir.
