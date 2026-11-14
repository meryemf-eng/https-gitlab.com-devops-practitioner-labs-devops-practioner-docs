# Git ve kaynak kod yönetimi

Bu dokümanlarda anlatılan hemen her şey bir Git deposunda yaşar: uygulama kodu, testler, Dockerfile'lar, pipeline tanımları, Terraform kodu, Kubernetes manifest'leri ve Helm chart'ları. Git'i sağlam öğrenmeden sonraki konuların hiçbiri yerine oturmaz. Bu dokümanda versiyon kontrolünün mantığını, Git'in iç modelini, branch ve merge stratejilerini ve GitHub ile GitLab üzerindeki iş birliği akışını ele alıyoruz.

## Versiyon kontrolü ve Git

Versiyon kontrolü, dosyalardaki değişiklikleri zaman içinde kayıt altına alma sürecidir. Her kayıt dört soruya cevap verir: ne değişti, ne zaman, kim değiştirdi ve neden. Kayıtlar projenin o anki tam bir fotoğrafıdır; istenen fotoğrafa dönülebilir, iki fotoğraf karşılaştırılabilir. Versiyon kontrolü birlikte çalışmayı, sürümleri saklamayı, yedeklemeyi ve bir hatanın ne zaman ve neden girdiğini bulmayı sağlar. Versiyon kontrolü olmadan geri dönüş, denetim ve otomasyon mümkün değildir.

Git ve GitHub sıkça karıştırılır. **Git**, değişiklikleri izleyen ve bilgisayarınızda internetsiz çalışan dağıtık bir versiyon kontrol aracıdır. **GitHub** ise Git kullanan, kodu çevrimiçi barındıran ve üzerine pull request, issue ve otomasyon gibi iş birliği araçları ekleyen bir platformdur. GitLab ve Bitbucket da aynı türden platformlardır.

İki tür versiyon kontrol sistemi vardır. SVN gibi **merkezi** sistemlerde tek bir sunucu tüm geçmişi tutar; geliştiriciler yalnızca bir çalışma kopyası alır ve sunucuya erişim yoksa commit yapamaz. Git gibi **dağıtık** sistemlerde ise her geliştirici tüm geçmişin bir kopyasını taşır; commit, geçmişe bakma ve branch açma internetsiz yapılabilir ve her kopya aynı zamanda bir yedektir. Git 2005'te Linux çekirdeğinin geliştirilmesi için yazıldı.

Bir Git deposunda yalnızca kaynak kodu değil, uygulamayı üreten, çalıştıran ve doğrulayan her şey sürümlenir: testler, build ve dağıtım script'leri, altyapı kodu, yapılandırma, dokümantasyon ve politikalar. İyi bir kural şudur: bir ortamı sıfırdan kurmak için gereken her şey depodan geri üretilebilmelidir. Depoya girmemesi gerekenler de bellidir: parolalar, token'lar, özel anahtarlar, müşteri verisi, build çıktıları ve büyük ikili dosyalar. Bir sır depoya bir kez girerse, sonradan silinse bile geçmişte kalır.

## Git'in üç alanı

Git'i anlamanın anahtarı üç alanı ayırt etmektir.

**Working tree**, dosyaları editörde değiştirdiğiniz yerdir. **Staging area**, `git add` ile bir sonraki commit'e girecek değişiklikleri seçtiğiniz yerdir. **Commit**, seçilen değişikliğe bir kimlik ve açıklama verilerek oluşturulan kayıttır. Staging alanının varlık sebebi, birbirinden bağımsız değişiklikleri ayrı commit'lere bölebilmektir: beş dosyada iki ayrı konu üzerinde çalıştıysanız, bunları iki anlamlı commit olarak kaydedebilirsiniz. `git diff` working tree ile staging arasındaki farkı, `git diff --staged` ise staging ile son commit arasındaki farkı gösterir.

Commit'lerin üzerindeki yazar bilgisi `git config` ile ayarlanır ve bir kimlik doğrulaması değildir; herkes istediği ismi yazabilir. Bir commit'in gerçekten kimden geldiğini doğrulamak için imzalı commit gerekir.

**.gitignore** dosyası, Git'in takip etmemesi gereken dosyaları listeler: ortam dosyaları, anahtarlar, bağımlılık klasörleri, loglar. Önemli bir ayrıntı, .gitignore'un yalnızca henüz takip edilmeyen dosyaları etkilemesidir; daha önce commit'lenmiş bir dosya takip edilmeye devam eder ve `git rm --cached` ile takipten çıkarılması gerekir.

İyi bir commit mesajı, gelecekteki okuyucuya "ne ve neden" sorusunun cevabını verir. **Conventional Commits** standardı ortak bir biçim önerir: bir tip, isteğe bağlı bir kapsam ve kısa bir açıklama; örneğin `fix(cart): boş sepette toplamın NaN olmasını engelle`. Tipler sürüm numarasıyla da ilişkilidir: `fix` yama numarasını, `feat` küçük sürüm numarasını, ünlem işaretiyle belirtilen geriye uyumsuz bir değişiklik ise büyük sürüm numarasını artırır. Bu sayede değişiklik günlüğü ve sürüm numarası otomatik üretilebilir. Temel kural, bir commit'in tek bir mantıksal değişiklik içermesidir.

## Branch ve merge

Git'te bir branch, geçmişin bir kopyası değil, bir commit'i gösteren hafif bir işaretçidir. Yeni bir branch açmak geçmişi kopyalamaz; bu yüzden anlık ve maliyetsizdir. Commit yaptıkça bulunulan branch'in işaretçisi ileri hareket eder. **HEAD** ise o anda nerede olduğunuzu gösteren özel işaretçidir. Branch'in bu kadar ucuz olması, her iş için yeni bir branch açmayı doğal bir alışkanlık hâline getirir.

İki tür birleştirme vardır. Hedef branch'te siz çalışırken hiç yeni commit olmadıysa Git yalnızca işaretçiyi ileri sarar; buna **fast-forward** denir ve yeni bir birleştirme commit'i gerekmez. Geçmiş ayrıştıysa, yani siz çalışırken hedef branch'e başkaları da commit eklediyse, Git iki ucu ve ortak atayı karşılaştırarak birleştirir ve bir **merge commit** oluşturur; buna **three-way merge** denir.

Git çoğu birleştirmeyi kendisi yapar, ama iki branch aynı dosyanın aynı satırlarını farklı biçimde değiştirdiyse karar insana kalır. Bu durumda **merge conflict** oluşur. Çakışan dosyada iki versiyon özel işaretlerle gösterilir; hangisinin kalacağına karar verilir, işaretler silinir, dosya stage edilir ve birleştirme tamamlanır. Çözümden sonra uygulamayı çalıştırmak ve testleri geçirmek şarttır; derlenen kod doğru kod demek değildir.

**Rebase**, birleştirmenin alternatifidir. Merge geçmişi olduğu gibi korur ve birleştirme bir merge commit'iyle görünür. Rebase ise feature branch'teki commit'leri hedef branch'in ucuna yeniden yazar ve doğrusal bir geçmiş üretir. İki yöntem aynı kodu üretir; fark geçmişin görünümündedir.

Rebase'in altın kuralı, başkalarıyla paylaşılmış, yani uzak depoya gönderilmiş commit'lerin rebase edilmemesidir. Rebase commit'lerin kimliklerini değiştirir ve ekip arkadaşlarının geçmişi sizinkinden kopar. Yaygın ekip kuralı, kişisel branch'te rebase, ortak branch'te merge kullanmaktır.

## Branching stratejileri

İki temel branching stratejisi vardır. **Git Flow**, uzun ömürlü develop, release ve hotfix branch'leri kullanır; planlı, takvime bağlı sürümler için tasarlanmıştır. **Trunk-based development** ise kısa ömürlü branch'ler ve günde birkaç kez ana dala birleştirmeye dayanır. CI/CD için önerilen model budur ve DORA araştırmaları trunk-based çalışmayı yüksek teslimat performansıyla ilişkilendirir. Bunun sebebi küçük değişiklik ilkesidir: Git Flow'da bir feature branch haftalarca yaşayabilir ve birleştirme günü büyük çakışmalara dönüşür; trunk-based modelde branch'ler bir iki gün yaşar ve çakışmalar küçük kalır. **GitHub Flow** ise ikisinin arasında, ana dalın her zaman yayına hazır olduğu ve her işin bir branch ile pull request üzerinden ilerlediği basit bir modeldir.

## Değişikliği geri almak

"Bir şeyi bozdum" durumunda doğru komut, değişikliğin nerede olduğuna bağlıdır. Working tree'deki bir değişiklikten vazgeçmek için `git restore`, yanlışlıkla stage edilen bir dosyayı geri çıkarmak için `git restore --staged`, henüz paylaşılmamış son commit'in mesajını düzeltmek için `git commit --amend`, son commit'i değişiklikleri koruyarak geri almak için `git reset --soft` kullanılır. `git reset --hard` commit'lenmemiş bütün değişiklikleri siler ve geri dönüşü yoktur.

Paylaşılmış bir geçmişte ise kural farklıdır: paylaşılmış geçmiş yeniden yazılmaz, düzeltilir. Ana dala gönderilmiş hatalı bir commit, `git revert` ile geri alınır; revert geçmişi silmez, hatalı commit'in tersini yapan yeni bir commit ekler. `git reflog` ise HEAD'in son konumlarını gösterdiği için, yanlışlıkla yapılan bir reset'ten sonra kaybolduğu sanılan commit'leri bulmayı sağlar.

## Uzak depolarla çalışmak

Uzak depo (remote), projenin bulutta duran kopyasıdır. **Clone** uzak depoyu bilgisayara kopyalar. **Fork**, başkasının deposunun kendi hesabınızdaki kopyasıdır ve bir Git komutu değil, platform özelliğidir. **Push** yerel commit'leri uzağa gönderir. **Fetch** yenilikleri yalnızca indirir; **pull** ise indirir ve birleştirir, yani fetch ile merge'in toplamıdır. Önce fetch ile neyin geldiğine bakmak sürprizleri önler. `git pull --ff-only` ise yalnızca fast-forward mümkünse ilerler; geçmiş ayrıştıysa durur ve kendiliğinden bir merge commit'i üretmek yerine kararı size bırakır.

Uzak depoya iki yoldan erişilir: bir anahtar çifti ile **SSH** ya da bir erişim token'ı ile **HTTPS**. GitHub artık HTTPS'te parola kabul etmez; parola yerine yetkisi ve süresi sınırlandırılmış bir kişisel erişim token'ı kullanılır. Token'lar ve özel anahtarlar commit mesajlarına, depolara ve ekran görüntülerine girmemelidir.

## GitHub, GitLab ve pull request

GitHub ve GitLab aynı Git'i kullanır, kavramları birebir eşleşir ama isimleri farklıdır. Üst düzey alan GitHub'da organization, GitLab'da group'tur ve GitLab iç içe alt gruplara izin verir. Depo GitHub'da repository, GitLab'da project olarak adlandırılır. Değişiklik önerisi GitHub'da pull request, GitLab'da merge request'tir. CI/CD tarafında GitHub Actions ve GitLab CI karşılık gelir. GitLab'ın önemli bir farkı, kurumun kendi sunucusuna kurulabilmesidir.

**Pull request**, bir onay formu değil, bir konuşmadır. İyi bir pull request küçüktür, tek bir konu içerir, ne değiştiğini, neden değiştiğini ve nasıl doğrulandığını açıklar ve gönderilmeden önce yazarı tarafından bir kez okunmuştur. Otomatik kontroller geçmeden inceleme istenmez. İnceleyen taraf kodun biçiminden çok davranışını sorgular, zorunlu ve isteğe bağlı önerilerini ayırır ve hızlı geri döner; bekleyen bir pull request, değer akışındaki en pahalı beklemelerden biridir. Küçük değişiklikler ve hızlı kod incelemesi, DORA araştırmalarında teslimat performansını doğrudan artıran yetenekler arasındadır.

Ana dal platform kurallarıyla korunur, çünkü güven insana değil kurala dayanmalıdır. **Branch protection** ana dala doğrudan ve zorla gönderimi yasaklar ve değişikliklerin yalnızca pull request ile gelmesini sağlar. En az bir ya da iki onay zorunlu tutulabilir ve yeni bir değişiklik geldiğinde eski onay düşer. CI ve testler başarılı olmadan birleştirme engellenir. **CODEOWNERS** dosyası, dosya yoluna göre otomatik inceleyici atar; örneğin altyapı klasörüne dokunan her değişikliğe platform ekibi eklenir. İmzalı commit ve doğrusal geçmiş de zorunlu tutulabilir.

Son olarak **tag**, bir commit'e `v1.4.0` gibi bir isim verir. Açıklamalı tag'ler `git tag -a` ile oluşturulur ve normal push ile gönderilmez; ayrıca gönderilmeleri gerekir. GitHub'daki release ise bir tag'in üzerine sürüm notlarını ve yayın dosyalarını ekler ve bir dağıtım pipeline'ını tetiklemek için kullanılabilir.

> Git dağıtık bir versiyon kontrol sistemidir; working tree, staging ve commit ayrımı onu anlamanın anahtarıdır. Branch ucuzdur, paylaşılmış geçmiş yeniden yazılmaz, ana dal kurallarla korunur ve küçük pull request'ler hızlı akışın temelidir.
