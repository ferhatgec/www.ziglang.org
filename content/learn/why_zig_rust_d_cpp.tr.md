---
title: "C++, D ve Rust varken Niye Zig?"
mobile_menu_title: "Niye Zig kullanalım..."
toc: true
---


## Gizli kontrol akışı yok

Eğer Zig kodu bir fonksiyonu çağırmak için farklı yere gidiyor gibi görünmüyorsa, öyle değildir. Bu da, aşağıdaki kodun yalnızca `foo()` ve sonra `bar()`'ı çağırdığından emin olacağınız anlamına gelir, ve bu türlerin herhangi birini bilmenize gerek kalmadan derleyici tarafından garanti edilir.

```zig
var a = b + c.d;
foo();
bar();
```

Gizli kontrol akış örnekleri:

* D `@property` fonksiyonlarına sahiptir, bunlar alan erişimine benzeyen çağrılabilir metodlardır. Yani üstteki örneğe göre, `c.d` fonksiyon çağırabilir.
* C++, D ve Rust operatör aşırı yükleme desteğine sahiptir, yani `+` operatörü bir fonksiyon belirtebilir ve onu çağırabilir.
* C++, D ve Go throw/catch'e sahiptir, yani `foo()` istisna gönderebilir ve `bar()`'ın çağrılmasına önayak olabilir. (Tabii ki, Zig'te bile `foo()` parçacıkları bekletebilir bu da işi kilitleyebilir ve `bar()`'ın çağrılmasını engelleyebilir, yine de bu Turing-complete dillerde olan bir durumdur.)

Bu tasarım tercihinin amacı okunabilirliği artırmaktır.

## Gizli bellek tahsisi yok

Yığın bellek tahsisi söz konusu olduğunda Zig'in müdahaleci bir yaklaşımı vardır. Diğer dillerdeki gibi yığın bellek tahsisi için `new` anahtar sözcüğü yoktur
(mesela dizgi birleştirme operatörü[1]).

Bütün yığın tasarım konsepti kütüphane ve uygulama kodu tarafından yönetilir, dil tarafından değil.

Gizli bellek tahsisi örnekleri:

* Go'nun `defer` fonksiyon-yerel yığınına bellek tahsis eder. 
  Bu kontrol akışının çalışması için sezgisel olmayan bir yol olmasının yanı sıra, 
  bir döngü içinde `defer` kullanırsanız yetersiz bellek hatalarına neden olabilir.
* C++ eşyordamları (coroutine), bir eşyordamı çağırmak için yığın belleği tahsis eder.
* Go'da, fonksiyon çağrısı, yığın bellek tahsisine neden olabilir çünkü goroutine'ler, çağrı yığını yeterince derinleştiğinde yeniden boyutlandırılan küçük yığınları bellekten tahsis eder. 
* Ana Rust standart kütüphane API'leri, yetersiz bellek koşullarında panik meydana getirir
 ve bellek tahsis eden parametreleri kabul eden alternatif API'ler sonrasında düşünülür. 
 (daha fazlası için [rust-lang/rust#29802](https://github.com/rust-lang/rust/issues/29802)'a bakın).

Çöp toplayıcı, temizleme tarafındaki kanıtları gizlediğinden, neredeyse tüm çöp toplama desteği olan diller 
etrafa saçılmış gizli ayrımlara sahiptir.

Gizli bellek tahsislerinin ana problemi bir kodun *yeniden kullanılabilirliğine* engel olmasıdır.
Basitçe söylemek gerekirse, bellek tahsisinin yan etkisine sahip olmamak için kontrol akışına ve 
fonksiyon çağrılarına güvenilmesi gereken durumlar vardır, bu nedenle bir programlama dili yalnızca bu garantiyi 
gerçekçi bir şekilde sağlayabilirse bu kullanım durumlarında kullanılabilir.

Zig'de yığın tahsisini sağlayan ve bunlarla birlikte çalışan standart kütüphane özellikleri vardır, 
ancak bunlar isteğe bağlı standart kütüphane özellikleridir ve dilin kendisinde yerleşik değildir. 
Bir yığın tahsisini asla başlatmazsanız, programınızın yığın tahsis etmeyeceğinden emin olabilirsiniz.

Yığın belleği tahsis etmesi gereken her standart kütüphane özelliği, bunu yapmak için bir `Allocator` parametresini kabul eder. 
Bu, Zig standart kütüphanesinin bağımsız hedefleri desteklediği anlamına gelir. Mesela, `std.ArrayList` v e`std.AutoHashMap` metal programlama için kullanılabilir!

Özel bellek tahsis ediciler, manuel bellek yönetimini çocuk oyuncağı haline getirir. 
Zig, kullanım sonrası bellek boşaltma ve çift bellek boşaltma karşısında bellek güvenliğini koruyan bir hata 
ayıklama ayırıcısına sahiptir. Bellek sızıntılarının yığın izlerini otomatik olarak algılar ve yazdırır. 
Her bir tahsisi bağımsız olarak yönetmek yerine istediğiniz sayıda tahsisi tek bir pakette toplayabilmeniz ve 
hepsini bir kerede serbest bırakabilmeniz için bir tahsis edici vardır. Özel amaçlı bellek tahsis ediciler, 
herhangi bir belirli uygulamanın gereksinimleri için performansı veya bellek kullanımını iyileştirmek için kullanılabilir.

[1]: Aslında bir dizgi ekleme operatörü vardır (genellikle bir dizi ekleme operatörü), ancak yalnızca derleme zamanında çalışır, bu nedenle herhangi bir çalışma zamanı yığın bellek tahsisi yapmaz.

## Standart olmayan bir kitaplık için birinci sınıf destek

Yukarıda bahsedildiği gibi, Zig tamamen isteğe bağlı bir standart kitaplığa sahiptir. 
Her std lib API, yalnızca onu kullanırsanız derlenir.
Zig, libc'ye bağlama veya bağlamama konusunda eşit desteğe sahiptir. Zig, metal ve yüksek performanslı geliştirmeye her daim dosttur.

## Kütüphaneler için Taşınabilir bir Dil

Programlamanın kutsal kısımlarından biri kodun yeniden kullanılabilirliğidir. Ne yazık ki pratikte kendimizi tekerleği defalarca yeniden icat ederken buluyoruz. Haklısınız.

 * Bir uygulamanın gerçek zamanlı gereksinimleri varsa, çöp toplama veya diğer herhangi bir deterministik olmayan davranış kullanan herhangi bir kitaplık, bağımlılık olarak odak dışı tutulur.
 * Bir dil, hataları göz ardı etmeyi çok kolaylaştırıyorsa ve bu nedenle bir kitaplığın hataları doğru bir şekilde işlediğini doğrulamayı zorlaştırıyorsa, ilgili tüm hataların doğru bir şekilde ele alındığını bilerek kitaplığı yok saymak ve yeniden uygulamak mantıklı olabilir. Zig, programcının yapabileceği en tembel şeyin hataları doğru bir şekilde ele alması olacak şekilde tasarlanmıştır ve bu nedenle, bir kitaplığın hataları düzgün bir şekilde göstereceğinden makul ölçüde emin olabilir.
 * Şu anda C'nin en çok yönlü ve taşınabilir dil olduğu pragmatik olarak doğrudur. C koduyla etkileşime girme yeteneğine sahip olmayan herhangi bir dil, anlaşılmazlık riski taşır. Zig, harici fonksiyonlar için C ABI'ye uymayı kolaylaştırarak ve uygulamalardaki yaygın hataları önleyen güvenlik ve dil tasarımı sunarak kitaplıklar için yeni taşınabilir dil olmaya çalışıyor.

## Halihazırda Var Olan Projeler için Paket Yöneticisi ve İnşa Sistemi

Zig programlama dilidir, ancak yanında geleneksel C/C++ içeren projeler için bile kullanışlı inşa sistemi ve paket yöneticisiyle gelir. 

Sadece C ya da C++ kodu yerine Zig kodu yazmazsınız, Zig'i autotools, cmake, make, scons, ninja gibi araçlar yerine de kullanırsınız. Ve bunun da ötesinde, yerel bağımlılıklar için paket yöneticisi de sağlayacaktır. İnşa sistemi, C veya C++ dilinde yazılan projeleriniz için de uyumlu olması için yazılmıştır.

apt-get, pacman, homebrew vs. gibi paket yöneticileri son kullanıcı için birer enstrüman gibidir, fakat geliştiricilerin ihtiyaçları karşısında yetersiz kalabilirler. Dile özgü bir paket yöneticisi, katkıda bulunanların olmaması ile çok sayıda olması arasındaki fark olabilir. Açık kaynak projeler için, projenin inşa edilmesini sağlamanın zorluğu, potansiyel katkıda bulunanlar için büyük bir engeldir. C/C++ projeleri için, özellikle paket yöneticisinin olmadığı Windows'ta bağımlılıklara sahip olmak çok sorunlu olabilir. Zig'i oluştururken bile, potansiyel katkıda bulunanların çoğu LLVM bağımlılığı konusunda zor anlar yaşamaktadır. Zig, projelerin yerel kitaplıklara doğrudan bağlı olması için bir yol sunuyor (olacak). Hangi sistemin kullanıldığına ve hangi platformun hedeflendiğinden bağımsız olarak.

Zig, bir projenin inşa sistemini, aynı zamanda paket yöneticisi ve dolayısıyla diğer C kitaplıklarına gerçekten bağımlı olma yeteneği de sağlayan, projeleri oluşturmak için bildirimsel bir API kullanarak makul bir dille değiştirmeyi teklif ediyor. Bağımlılıklara sahip olma yeteneği, daha yüksek düzeyde soyutlamalara ve dolayısıyla yeniden kullanılabilir yüksek düzeyli kodun artmasına olanak tanır.

## Basitlik

C++, Rust ve D çok fazla özelliğe sahiptir, bunlar da uygulamanızın gerçek anlamını uzaklaştırabilir. Kullanan kişi, uygulamasında hata arayacağı yerde zorunlu olarak programlama dili bilgisinde hata aramaya başlar.

Zig ne makroya ne de metaprogramlamaya sahiptir, yine de karmaşık programları temiz, tekrar eden kod halinde olmadan bile ifade edebilir. Rust'ta bile derleyici içinde spesifik kullanım amaçlı önceden yazılmış makrolar vardır. Mesela `format!` örnek verilebilir. Zig'te ise, eşdeğer fonksiyon standart kütüphanede yazılmıştır, derleyiciye özel durum kodları yoktur.

## Araç seti

Zig [indirme bölümünden](/download/) indirilebilir. Zig Linux, Windows, macOS ve FreeBSD için arşivler sunar. Aşağıdakiler, arşivden neler elde edeceğinizi anlatır:

* tek bir arşivi indirir, kurarsınız. sisteme özgü özelleştirmelere gerek yoktur
* çalışma anı bağımlılıkları yoktur çünkü statik derlenir
* çoğu büyük platform için derin optimizasyon ve destek sağlayan iyi desteklenen LLVM altyapısını kullanmaktadır 
* çoğu büyük platforma çapraz derleme desteği birlikte gelir
* libc'nin kaynak koduyla birlikte dağıtılır bu da, gerektiğinde desteklenen platformlar için dinamik derlenir
* önbellek destekli inşa sistemiyle gelir
* libc desteğiyle C ve C++ koduna derlenir 
