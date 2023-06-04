---
title: Hadi Başlayalım
mobile_menu_title: "Hadi Başlayalım"
toc: true
---

## Etiketlenmiş sürümler mi yoksa gecelik inşalar mı?
Zig henüz v1.0'a ulaşmadı ve mevcut sürüm döngüsü LLVM'in yeni sürümlerine bağlı, yaklaşık 6 ay sürer.

Diğer açıdan, **Zig sürümleri birbirlerinden çok uzaktırlar ve mevcut geliştirme hızı göz önüne alındığında eski sürümler pratik açıdan bayatlamış gibi düşünülebilir.**.

Zig'in etiketlenmiş sürümlerini kullanmanızda bir sakınca yoktur, Zig'i sevdiyseniz ve daha fazla derine inmek istiyorsanız;
**gecelik inşaları kullanmanızı tavsiye ederiz** çünkü bu şekilde kullanırsanız daha hızlı yardıma kavuşursunuz:
[ziglearn.org](https://ziglearn.org) gibi topluluk ve siteler üstteki sebeplerden ötürü master dalını takip eder.

İyi haber şu ki, bir Zig sürümünden diğerine geçiş çok kolaydır, aynı şekilde birçok Zig sürümünü sisteminizde tutabilirsiniz: Zig sürümleri birbirinden bağımsız, istediğiniz yerde tutabileceğiniz arşivlerdir.

## Zig'i Yüklemek
### Direk indirme
Zig'i yüklemenin en basit yoludur: platformunuza uygun Zig paketini [Downloads](/download) sayfasından indirin,
herhangi bir dizine çıkartın ve `PATH` çevre değişkeninize ekleyin ki `zig`'i herhangi bir yerden çalıştırabilin.

#### Windows'ta PATH değişkenini ayarlama
Ayarlamak için Windows'ta şu kod bloklarından **sadece birisini** Powershell kabuğunda çalıştırın.
Sistem seviyesinde uygulamak isteyip istemediğinizi seçin (Powershell'i yönetici yetkileriyle çalıştırmalısınız)
ya da sadece mevcut kullanıcı için, **kodu Zig'in bulunduğu konumu gösterecek şekilde değiştirdiğinizden emin olun**.
`C:`'den önceki `;` yazım hatası değildir.

Sistem sevisyesinde (**yönetici** Powershell):
```
[Environment]::SetEnvironmentVariable(
   "Path",
   [Environment]::GetEnvironmentVariable("Path", "Machine") + ";C:\your-path\zig-windows-x86_64-your-version",
   "Machine"
)
```

Kullanıcı seviyesinde (Powershell):
```
[Environment]::SetEnvironmentVariable(
   "Path",
   [Environment]::GetEnvironmentVariable("Path", "User") + ";C:\your-path\zig-windows-x86_64-your-version",
   "User"
)
```
Yaptıktan sonra, Powershell kabuğunuzu yeniden başlatın.

#### Linux, macOS, BSD'de PATH değişkenini ayarlama
Zig yürütülebilir dosyanızın bulunduğu konumu PATH çevre değişkenine ekleyin.

Bu genellikle kabuğunuzun başlatma betiğine export satırı ekleyerek yapılabilir. (`.profile`, `.zshrc`, ...)

```bash
export PATH=$PATH:~/path/to/zig
```
Yaptıktan sonra, ya `source` komutunu kullanın ya da kabuğunuzu yeniden başlatın.

### Paket yöneticileri
#### Windows
Zig [Chocolatey](https://chocolatey.org/packages/zig)'de mevcuttur.
```
choco install zig
```

#### macOS

**Homebrew**  
En son etiketlenmiş sürüm:
```
brew install zig
```

Git master dalındaki son inşa:
```
brew install zig --HEAD
```

**MacPorts**
```
port install zig
```
#### Linux
Zig çoğu Linux paket yöneticisinde bulunur. [Burada](https://github.com/ziglang/zig/wiki/Install-Zig-from-a-Package-Manager)
güncel listeyi bulabilirsiniz ancakl unutmayın ki bazı paketler güncel olmayan Zig sürümlerinde olabilir.

### Kaynaktan derlemek
[Burada](https://github.com/ziglang/zig/wiki/Building-Zig-From-Source) 
Linux, macOS ve Windows'ta Zig'i nasıl kaynak kodundan derleyebileceğiniz hakkında daha fazla bilgiye ulaşabilirsiniz.

## Tavsiye Edilen Araçlar
### Söz Dizimi Renklendiricleri ve Dil Sunucu Protokolü
Çoğu yazı editörü Zig için söz dizimi renklendirmesine sahiptir.
Bazı editörler paket halinde, bazıları eklenti halinde kullanıma sunar.

Eğer Zig ve editörünüz arasındaki derin integrasyonla ilgileniyorsanız, [zigtools/zls](https://github.com/zigtools/zls)'a bir bakın.

Eğer mevcut olan başka bir şeyle ilgileniyorsanız [Araçlar](../tools/) bölümüne bir bakın.

## Hello World Kodu Çalıştırma
Eğer yükleme aşamalarını doğru yaptıysanız, Zig derleyicisini kabuğunuzdan çalıştırabiliyor olmanız gerekir.
Hadi ilk Zig programınızı oluşturarak bunu deneyelim!

Proje dizininize gidin ve çalıştırın:
```bash
mkdir hello-world
cd hello-world
zig init-exe
```

Şu çıktıyı almanız gerekir:
```
info: Created build.zig
info: Created src/main.zig
info: Next, try `zig build --help` or `zig build run`
```

`zig build run`'ı çalıştırmak, yürütülebilir dosyayı derlemeli ve onu çalıştırmalıdır, şunu görmelisiniz:
```
info: All your codebase are belong to us.
```

Tebrikler, sorunsuz çalışan Zig kurulumuna sahipsiniz!  

## Sonraki aşamalar
**[Öğrenelim](../) bölümündeki faydalı kaynaklara göz atın**, Zig versiyonunuzun Dokümantasyonla eşleştiğinden emin olun.
(not: gecelik inşalar için `master` dokümantasyonlarını kullanın) ve [ziglearn.org](https://ziglearn.org)'u okumaya bir şans verin.

Zig genç bir proje ve maalesef henüz çok geniş dokümantasyon ve öğretici materyalleri herkese ulaştırabilecek ve oluşturabilecek kapasiteye sahip değiliz.
Eğer düşünürseniz, [halihazırda var olan Zig topluluklarına katılmayı düşünebilirsiniz](https://github.com/ziglang/zig/wiki/Community)
böylece takıldığınız yerlerde insanlara hem yardım edebilir hem de onlardan yardım alabilirsiniz, ayrıca [Zig SHOWTIME](https://zig.show) gibi oluşumlara da bir göz atabilirsiniz.

Sonuç olarak, eğer Zig'i sevdiyseniz ve geliştirme sürecinde yardım etmek istiyorsanız, [Zig Software Foundation'a bağış yapmayı düşünebilirsiniz](../../zsf)
<img src="/heart.svg" style="vertical-align:middle; margin-right: 5px">.
