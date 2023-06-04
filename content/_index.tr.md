---
title: Home
mobile_menu_title: "Home"
---
{{< slogan get_started="Hadi Başlayalım" docs="Dokümantasyon" notes="Değişiklikler" lang="tr" >}}
Zig, genel kullanım amaçlı, **güçlü**, **ideal** ve **tekrar kullanılabilir** yazılımlar için uygun bir programlama dili ve araç kitidir.  
{{< /slogan >}}

{{< flexrow class="container" style="padding: 20px 0; justify-content: space-between;" >}}
{{% div class="features" %}}

# ⚡ Basit Bir Dil
Programlama dili bilginizde değil, uygulamanızda hata aramanıza olanak sağlar.

- Gizli kontol akışları yok.
- Gizli bellek tahsisi.
- Ön işleyici ve makrolar yok. 

# ⚡ Comptime
Metaprogramlamaya derleme zamanı kod çalıştırma ve tembel değerlendirme tabanlı yeni yaklaşım.

- Derleme zamanında herhangi bir fonksiyonu çağırın.
- Çalışma zamanında ek yük olmadan türleri değerler olarak manipüle edin.
- Comptime hedef mimariyi taklit eder.

# ⚡ Zig ile Sürdürün
C/C++/Zig kod tabanınızı sırayla iyileştirin.

- Zig'i harici bağımlılık olmadan kullanın, dahili C/C++ derleyicisi çapraz derlemeyi kullanıma hazır olarak destekler.
- Tüm platformlarda tutarlı bir geliştirme ortamı için `zig build`'ten yararlanın.
- C/C++ projelerine Zig derleme takımı ekleyin; çapraz dil bağlama süresi optimizasyonu varsayılan olarak açıktır.

{{% flexrow style="justify-content:center;" %}}
{{% div %}}
<h1>
    <a href="learn/overview/" class="button" style="display: inline;">Derince bir bakış</a>
</h1>
{{% /div %}}
{{% div  style="margin-left: 5px;" %}}
<h1>
    <a href="learn/samples/" class="button" style="display: inline;">Daha fazla hazır kod</a>
</h1>
{{% /div %}}
{{% /flexrow %}}
{{% /div %}}
{{< div class="codesample" >}}

{{% zigdoctest "assets/zig-code/index.zig" %}}

{{< /div >}}
{{< /flexrow >}}


{{% div class="alt-background" %}}
{{% div class="container"  style="display:flex;flex-direction:column;justify-content:center;text-align:center; padding: 20px 0;" title="Topluluk" %}}

{{< flexrow class="container" style="justify-content: center;" >}}
{{% div style="width:25%" %}}
<img src="/ziggy.svg" style="max-height: 200px">
{{% /div %}}

{{% div class="community-message" %}}
# Zig topluluğu merkeziyetsizdir 
İsteyen herkez kendi topluluğunu oluşturmakta ve sürdürmekte özgürdür.
"Resmi" ya da "resmi olmayan" diye bir anlayış yok, fakat, her topluluğun kendine has kuralları ve moderatörleri vardır.

<div style="">
<h1>
	<a href="https://github.com/ziglang/zig/wiki/Community" class="button" style="display: inline;">Tüm Toplulukları gör</a>
</h1>
</div>
{{% /div %}}
{{< /flexrow >}}
<div style="height: 50px;"></div>

{{< flexrow class="container" style="justify-content: center;" >}}
{{% div class="main-development-message" %}}
# Dilin geliştirilmesi
Zig deposu [https://github.com/ziglang/zig](https://github.com/ziglang/zig)'de bulunabilir. Bu depoda ayrıca bir hata takibi ve deneysel özellikleri tartışma sistemi de vardır.  
Katkıda bulunanların Zig'in [Davranış Kurallarına](https://github.com/ziglang/zig/blob/master/.github/CODE_OF_CONDUCT.md) uymaları beklenmektedir.
{{% /div %}}
{{% div style="width:40%" %}}
<img src="/zero.svg" style="max-height: 200px">
{{% /div %}}
{{< /flexrow >}}
{{% /div %}}
{{% /div %}}


{{% div class="container" style="display:flex;flex-direction:column;justify-content:center;text-align:center; padding: 20px 0;" title="Zig Software Foundation" %}}
## ZSF 501(c)(3) kar amacı gütmeyen bir kuruluştur.

Zig Software Foundation, Andrew Kelley tarafından 2020'de kar amacı gütmeyen bir kuruluş olarak kuruldu. Andrew Kelley, dilin gelişimini desteklemek amacıyla Zig'i oluşturmuştur. Şu anda, ZSF çok ve temel katkıda bulunan küçük bir azınlığa rekabetçi oranlarda ücretli iş sunabilir. Gelecekte daha fazla çok ve temel düzeyde katkıda bulunanları artırmayı umuyoruz.

Zig Software Foundation'ın giderleri bağışçılar tarafından karşılanmaktadır.

<h1>
	<a href="zsf/" class="button" style="display:inline;">Daha Fazlasını Öğrenin</a>
</h1>
{{% /div %}}


{{< div class="alt-background" style="padding: 20px 0;">}}
{{% div class="container" title="Destekçiler" %}}
# Kurumsal Destekçiler 
Listedeki kuruluşlar Zig Software Foundation'a direkt finansal destek sağlıyor.

{{% monetary-sponsor-logos %}}


# GitHub Sponsors
[Zig'e sponsor](zsf/) olan herkese teşekkürler! Proje, kurumsal desteklerden daha çok bireysel destekçiler tarafından idare edilmektedir. Aşağıdaki kişiler Zig'e $200/aylık ya da daha fazla bağışta bulundular:

{{< ghsponsors >}}

Bu bölüm her ayın başı güncellenmektedir.
{{% /div %}}
{{< /div >}}
