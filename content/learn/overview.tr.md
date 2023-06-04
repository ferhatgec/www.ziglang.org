---
title: "Derinlemesine Bakış"
mobile_menu_title: "Bakış"
toc: true
---
# Özellik Ön Gösterimleri
## Küçük, basit bir dil
Programlama dili bilginizde hata ayıklamayın uygulamanızda hata ayıklayın.

Zig'in bütün haliyle söz dizimi [500 satırlık PEG dil kuralı dosyasında](https://ziglang.org/documentation/master/#Grammar) bahsedilmiştir.

**Gizli kontrol akışı yok**, gizli bellek tahsisi yok, ön işleyici yok, makrolar yok. Zig kodu, bir fonksiyonu çağırmak için başka yere gidiyor gibi görünmüyorsa, öyle değildir. Bu, aşağıdaki kodun yalnızca `foo()` ve sonra `bar()` çağırdığından emin olabileceğiniz anlamına gelir ve bu, herhangi bir şeyin türünü bilmenize gerek kalmadan derleyici tarafından garanti edilir.

```zig
var a = b + c.d;
foo();
bar();
```

Examples of hidden control flow:

- D `@property` fonksiyonlarına sahiptir, bunlar alan erişimine benzeyen çağrılabilir metodlardır. Yani üstteki örneğe göre, `c.d` fonksiyon çağırabilir.
- C++, D ve Rust operatör aşırı yüklemesine sahiptir, yani `+` operatörü fonksiyon çağırabilir.
- C++, D ve Go throw/catch'e sahiptir, yani `foo()` istisna gönderebilir ve `bar()`'ın çağrılmasına önayak olabilir. (Tabii ki, Zig'te bile `foo()` parçacıkları bekletebilir bu da işi kilitleyebilir ve `bar()`'ın çağrılmasını engelleyebilir, yine de bu Turing-complete dillerde olan bir durumdur.)

Zig, tüm kontrol akışını özel olarak dil anahtar sözcükleri ve fonksiyon çağrılarıyla yöneterek kod bakımını ve okunabilirliği destekler.

## Performans and Güvenlik: İkisini de Seçin

Zig dört tane [inşa moduna](https://ziglang.org/documentation/master/#Build-Mode) sahiptir, hepsi birlikte kullanılabilir ve [kapsam ayrıntı düzeyinin](https://ziglang.org/documentation/master/#setRuntimeSafety) sonuna kadar eşleştirilebilir.

| Parametre                                                                          | [Debug](/documentation/master/#Debug) | [ReleaseSafe](/documentation/master/#ReleaseSafe) | [ReleaseFast](/documentation/master/#ReleaseFast) | [ReleaseSmall](/documentation/master/#ReleaseSmall) |
|------------------------------------------------------------------------------------|-------|-------------|-------------|--------------|
 Optimizasyonlar - hızı artırır, hata ayıklamaya zarar verir, derleme süresine zarar verir | | -O3 | -O3| -Os |
 Çalışma Zamanı Güvenlik Kontrolleri - tanımsız davranış yerine çöker | On | On | | |

İnşa moduna bakılmaksınız [Tam Sayı Taşmasının](https://ziglang.org/documentation/master/#Integer-Overflow) derleme anında nasıl göründüğüne bakın:

{{< zigdoctest "assets/zig-code/features/1-integer-overflow.zig" >}}

Güvenlik kontrolü yapılan inşalarda çalışma zamanında nasıl göründüğü:

{{< zigdoctest "assets/zig-code/features/2-integer-overflow-runtime.zig" >}}

Bu [yığın izleri](#stack-traces-on-all-targets), [bağımsız](https://andrewkelley.me/post/zig-stack-traces-kernel-panic-bare-bones-os.html) olanlar da dahil, tüm hedeflerde çalışır.

Zig ile kişi, güvenliğin etkinleştirildiği bir oluşturma moduna güvenebilir ve performans darboğazlarında güvenliği seçerek devre dışı bırakabilir. Örneğin, önceki örnek şu şekilde değiştirilebilir:

{{< zigdoctest "assets/zig-code/features/3-undefined-behavior.zig" >}}

Zig, [tanımsız davranışı](https://ziglang.org/documentation/master/#Undefined-Behavior) hem hata önleme hem de performans geliştirme için keskin bir araç olarak kullanır.

Performansı konuşursak, Zig C'den hızlıdır.

- Referans derleyici, son teknoloji optimizasyonlar için arka uçta LLVM kullanır.
- Diğer projelerin "Bağlama Süresi Optimizasyonu" dediği şeyi Zig otomatik olarak yapar.
- Yerel hedefler için, [çapraz derlemenin birinci sınıf bir kullanım durumu olması](#cross-compiling-is-a-first-class-use-case) sayesinde gelişmiş CPU özellikleri etkinleştirilir (-march=native).
- Dikkatle seçilmiş tanımsız davranış. Örneğin, Zig'de hem işaretli hem de işaretsiz tam sayılar, C'deki yalnızca işaretli tams ayıların aksine, taşma durumunda tanımsız davranışa sahiptir. Bu, [C'de bulunmayan optimizasyonları kolaylaştırır](https://godbolt.org/z/n_nLEU).
- Zig, [SIMD vektör türünü](https://ziglang.org/documentation/master/#Vectors) doğrudan ortaya koyarak taşınabilir vektörleştirilmiş kod yazmayı kolaylaştırır.

Lütfen Zig'in tamamen güvenli bir dil olmadığını unutmayın. Zig'in güvenlik hikayesini takip etmekle ilgilenenler şu konuları takip edebilir:

- [güvenlik kontrolünden geçemeyenler de dahil olmak üzere her türlü tanımlanmamış davranışı sıralayın](https://github.com/ziglang/zig/issues/1966)
- [Debug ve ReleaseSafe modlarını tamamen güvenli hale getirin](https://github.com/ziglang/zig/issues/2301)

## Zig, C'ye bağımlı olmak yerine C ile rekabet eder

Zig Standart Kütüphanesi libc ile bütünleşiktir, ancak ona bağımlı değildir. İşte Hello World:

{{< zigdoctest "assets/zig-code/features/4-hello.zig" >}}

`-O ReleaseSmall`, hata ayıklama sembolleri çıkarılmış, tek iş parçacıklı mod ile derlendiğinde, bu, x86_64-linux hedefi için 9.8 KiB'lik bir statik yürütülebilir dosya üretir:

```
$ zig build-exe hello.zig -O ReleaseSmall --strip --single-threaded
$ wc -c hello
9944 hello
$ ldd hello
  not a dynamic executable
```

Bir Windows derlemesi daha da küçüktür ve 4096 byte'a ulaşır:
```
$ zig build-exe hello.zig -O ReleaseSmall --strip --single-threaded -target x86_64-windows
$ wc -c hello.exe
4096 hello.exe
$ file hello.exe
hello.exe: PE32+ executable (console) x86-64, for MS Windows
```

## Sıra bağımsız üst düzey tanımlamalar 

Genel değişkenler gibi üst düzey tanımlamalar, sıra bağımsızdır ve tembelce analiz edilir. Genel değişkenlerin başlatma değerleri [derleme zamanında değerlendirilir](#compile-time-reflection-and-compile-time-code-execution).

{{< zigdoctest "assets/zig-code/features/5-global-variables.zig" >}}

## NULL işaretçiler yerine isteğe bağlı tür

Diğer programlama dillerinde, boş referanslar birçok çalışma zamanı istisnasının kaynağıdır ve hatta [bilgisayar biliminin en kötü hatası olmakla suçlanır](https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/).

Süslenmemiş Zig işaretçileri boş olamaz:

{{< zigdoctest "assets/zig-code/features/6-null-to-ptr.zig" >}}

Bununla birlikte, [opsiyonel tür](https://ziglang.org/documentation/master/#Optionals), ? ile ön eklenerek isteğe bağlı bir tür haline getirilebilir:

{{< zigdoctest "assets/zig-code/features/7-optional-syntax.zig" >}}

Opsiyonel bir değeri açmak için, varsayılan bir değer sağlamak üzere `orelse` kullanılabilir:

{{< zigdoctest "assets/zig-code/features/8-optional-orelse.zig" >}}

Diğer seçenek de if kullanmaktır:

{{< zigdoctest "assets/zig-code/features/9-optional-if.zig" >}}

 Aynı söz dizimi [while](https://ziglang.org/documentation/master/#while)'da da geçerlidir:

{{< zigdoctest "assets/zig-code/features/10-optional-while.zig" >}}

## Manuel bellek yönetimi

Zig ile yazılmış bir kitaplık her yerde kullanılabilir:

- [Masaüstü uygulamaları](https://github.com/TM35-Metronome/)
- Düşük gecikmeli sunucular
- [İşletim Sistemi çekirdekleri](https://github.com/AndreaOrru/zen)
- [Gömülü cihazlar](https://github.com/skyfex/zig-nrf-demo/)
- Gerçek zamanlı yazılım, örn. canlı performanslar, uçaklar, kalp pilleri
- [WebAssembly ile web tarayıcılarında veya diğer eklentilerde](https://shritesh.github.io/zigfmt-web/)
- Diğer programlama dilleri tarafından, C ABI kullanılarak

Bunu başarmak için, Zig programcıları belleği iyi yönetmeli ve bellek tahsis etme hatalarını halletmelidir.

Bu, Zig Standart Kütüphanesi için de geçerlidir. Bellek tahsis etmesi gereken tüm fonksiyonlar bir tahsis edici parametre kabul eder. 
Sonuç olarak, Zig Standart Kütüphanesi bağımsız hedef için bile kullanılabilir.

[Hata işlemeye yeni bir yaklaşıma ek olarak](#a-fresh-take-on-error-handling) Zig, yalnızca belleği değil tüm kaynak yönetimini basit ve kolayca doğrulanabilir hale getirmek için  [defer](https://ziglang.org/documentation/master/#defer) ve [errdefer](https://ziglang.org/documentation/master/#errdefer)sağlar.

For an example of defer, see Integration with C libraries without FFI/bindings. Here is an example of using errdefer:

Bir `defer` örneği için bkz. [FFI/bağlamalar olmadan C kitaplıklarıyla entegrasyon](#integration-with-c-libraries-without-ffibindings). İşte `errdefer` kullanımına bir örnek:

{{< zigdoctest "assets/zig-code/features/11-errdefer.zig" >}}

## Hata işlemeye yeni bir yaklaşım

Hatalar değerlerdir ve göz ardı edilemez:

{{< zigdoctest "assets/zig-code/features/12-errors-as-values.zig" >}}

Hatalar [catch](https://ziglang.org/documentation/master/#catch) ile yakalanabilir:

{{< zigdoctest "assets/zig-code/features/13-errors-catch.zig" >}}

[try](https://ziglang.org/documentation/master/#try) anahtar sözcüğü, `catch |err| return err` için bir kısayoldur:

{{< zigdoctest "assets/zig-code/features/14-errors-try.zig" >}}

Bunun bir yığın izlemesi değil, [Hata Dönüş İzlemesi](https://ziglang.org/documentation/master/#Error-Return-Traces) olduğunu unutmayın. Kod, bu izi bulmak için yığını çözmenin bedelini ödemedi.
Bir hatada kullanılan [switch](https://ziglang.org/documentation/master/#switch) anahtar sözcüğü, olası tüm hataların ele alınmasını sağlar:

{{< zigdoctest "assets/zig-code/features/15-errors-switch.zig" >}}

[unreachable](https://ziglang.org/documentation/master/#unreachable) anahtar sözcüğü, hiçbir hatanın olmayacağını belirtmek için kullanılır:

{{< zigdoctest "assets/zig-code/features/16-unreachable.zig" >}}

Bu, güvenli olmayan oluşturma modlarında [tanımsız davranışlara](#performance-and-safety-choose-two) neden olur, bu nedenle yalnızca kesinliği garanti edildiğinde kullandığınızdan emin olun.

### Tüm hedeflerde yığın izleme 

Bu sayfada gösterilen yığın izleri ve [hata döndürme izleri](https://ziglang.org/documentation/master/#Error-Return-Traces), [1. Kademe Destek](#tier-1-support) ve bazı [2. Kademe Destek](#tier-2-support) hedeflerinde çalışır. [Bağımsız olanlar](https://andrewkelley.me/post/zig-stack-traces-kernel-panic-bare-bones-os.html) bile!

Ek olarak, standart kitaplığın herhangi bir noktada bir yığın izi yakalama ve ardından bunu daha sonra standart hataya dökme yeteneği vardır:

{{< zigdoctest "assets/zig-code/features/17-stack-traces.zig" >}}

Devam eden [GeneralPurposeDebugAllocator project](https://github.com/andrewrk/zig-general-purpose-allocator/#current-status) projesinde bu tekniğin kullanıldığını görebilirsiniz.

## Yaygın veri yapıları ve fonksiyonları

Türler, derleme zamanında bilinmesi gereken değerlerdir:

{{< zigdoctest "assets/zig-code/features/18-types.zig" >}}

Yaygın bir veri yapısı, basitçe bir `tür` döndüren bir fonksiyondur:

{{< zigdoctest "assets/zig-code/features/19-generics.zig" >}}

## Derleme zamanı yansıması ve derleme zamanı kod yürütme

[@typeInfo](https://ziglang.org/documentation/master/#typeInfo) dahili fonksiyonu yansıtmayı sağlar:

{{< zigdoctest "assets/zig-code/features/20-reflection.zig" >}}

Zig Standart Kütüphanesi, biçimlendirilmiş yazdırmayı uygulamak için bu tekniği kullanır. [Küçük, basit bir dil](#small-simple-language) olmasına rağmen, Zig'in biçimlendirilmiş baskısı tamamen Zig'te yazılmıştır. Bu arada, C'de printf için derleme hataları derleyiciye sabit kodlanmıştır. Benzer şekilde, Rust'ta biçimlendirilmiş yazdırma makrosu derleyiciye sabit kodlanmıştır.

Zig, derleme zamanında fonksiyonları ve kod bloklarını da değerlendirebilir. Yaygın değişken oluşturulmaları gibi bazı bağlamlarda ifade, derleme zamanında dolaylı olarak değerlendirilir. Aksi takdirde, derleme zamanında [comptime](https://ziglang.org/documentation/master/#comptime) anahtar sözcüğüyle kod açıkça değerlendirilebilir. Bu, iddialarla birleştirildiğinde özellikle güçlü olabilir:

{{< zigdoctest "assets/zig-code/features/21-comptime.zig" >}}

## FFI/bağlamalar olmadan C kitaplıklarıyla entegrasyon

[@cImport](https://ziglang.org/documentation/master/#cImport), Zig'te kullanım için türleri, değişkenleri, fonksiyonları ve basit makroları doğrudan içe aktarır. Hatta satır içi fonksiyonları bile C'den Zig'e çevirir.

İşte [libsoundio](http://libsound.io/) kullanarak bir sinüs dalgası oluşturmanın bir örneği:

<u>sine.zig</u>
{{< zigdoctest "assets/zig-code/features/22-sine-wave.zig" >}}

```
$ zig build-exe sine.zig -lsoundio -lc
$ ./sine
Output device: Built-in Audio Analog Stereo
^C
```

Bu Zig kodu, [eşdeğer C kodundan önemli ölçüde daha basittir](https://gist.github.com/andrewrk/d285c8f912169329e5e28c3d0a63c1d8) ve daha fazla güvenlik korumasına sahiptir. Tüm bunlar, C başlık dosyasını doğrudan içe aktararak gerçekleştirilir - API bağlaması yoktur.

*Zig, C kitaplıklarını kullanmakta C'nin C kitaplıklarını kullanmasından daha iyidir.*

### Zig aynı zamanda bir C derleyicisidir.

İşte bir C kodu oluşturan Zig örneği:

<u>hello.c</u>
```c
#include <stdio.h>

int main(int argc, char **argv) {
    printf("Hello world\n");
    return 0;
}
```

```
$ zig build-exe hello.c --library c
$ ./hello
Hello world
```

Bunun hangi C derleyici komutunu çalıştırdığını görmek için `--verbose-cc`'yi kullanabilirsiniz:
```
$ zig build-exe hello.c --library c --verbose-cc
zig cc -MD -MV -MF zig-cache/tmp/42zL6fBH8fSo-hello.o.d -nostdinc -fno-spell-checking -isystem /home/andy/dev/zig/build/lib/zig/include -isystem /home/andy/dev/zig/build/lib/zig/libc/include/x86_64-linux-gnu -isystem /home/andy/dev/zig/build/lib/zig/libc/include/generic-glibc -isystem /home/andy/dev/zig/build/lib/zig/libc/include/x86_64-linux-any -isystem /home/andy/dev/zig/build/lib/zig/libc/include/any-linux-any -march=native -g -fstack-protector-strong --param ssp-buffer-size=4 -fno-omit-frame-pointer -o zig-cache/tmp/42zL6fBH8fSo-hello.o -c hello.c -fPIC
```

Komutu tekrar çalıştırırsanız herhangi bir çıktı oluşmayacağını ve anında duracağını unutmayın:
```
$ time zig build-exe hello.c --library c --verbose-cc

real	0m0.027s
user	0m0.018s
sys	0m0.009s
```

Bu, [Yapı Önbelleği İnşası](https://ziglang.org/download/0.4.0/release-notes.html#Build-Artifact-Caching) sayesindedir. Zig, .d dosyasını otomatik olarak ayrıştırır ve işi tekrarlamaktan kaçınmak için sağlam bir önbelleğe alma sistemi kullanır.

Zig yalnızca C kodunu derlemekle kalmaz, aynı zamanda Zig'i bir C derleyicisi olarak kullanmak için çok iyi bir neden vardır: [Zig, libc ile birlikte dağıtılır](#zig-ships-with-libc) 

### C kodunun bağımlı olması için fonksiyonları, değişkenleri ve türleri dışa aktarın

Zig'in birincil kullanım durumlarından biri, çağrılacak diğer programlama dilleri için C ABI ile bir kitaplığı dışa aktarmaktır. Fonksiyonların, değişkenlerin ve türlerin önündeki `export`anahtar sözcüğü, bunların kitaplık API'sinin parçası olmalarına neden olur:

<u>mathtest.zig</u>
{{< zigdoctest "assets/zig-code/features/23-math-test.zig" >}}

Statik bir kitaplık yapmak için:
```
$ zig build-lib mathtest.zig
```

Paylaşımlı bir kitaplık yapmak için:
```
$ zig build-lib mathtest.zig -dynamic
```

İşte [Zig İnşa Sistemi](#zig-build-system) ile bir örnek:

<u>test.c</u>
```c
#include "mathtest.h"
#include <stdio.h>

int main(int argc, char **argv) {
    int32_t result = add(42, 1337);
    printf("%d\n", result);
    return 0;
}
```

<u>build.zig</u>
{{< zigdoctest "assets/zig-code/features/24-build.zig" >}}

```
$ zig build test
1379
```

## Çapraz derleme birinci sınıf bir kullanım durumudur

Zig, [Destek Tablosundaki](#support-table) herhangi bir hedef için [3. Kademe Destek](#tier-3-support) veya daha iyisi için inşa edebilir. Hiçbir "çoklu platform araç zinciri" veya buna benzer bir şeyin kurulmasına gerek yoktur. İşte Hello World:

{{< zigdoctest "assets/zig-code/features/4-hello.zig" >}}

Şimdi, x86_64-windows, x86_64-macosx ve aarch64v8-linux için inşa edin:
```
$ zig build-exe hello.zig -target x86_64-windows
$ file hello.exe
hello.exe: PE32+ executable (console) x86-64, for MS Windows
$ zig build-exe hello.zig -target x86_64-macosx
$ file hello
hello: Mach-O 64-bit x86_64 executable, flags:<NOUNDEFS|DYLDLINK|TWOLEVEL|PIE>
$ zig build-exe hello.zig -target aarch64v8-linux
$ file hello
hello: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped
```

Bu, herhangi bir [3+. Kademe](#tier-3-support) hedef için, herhangi bir [3+. Kademe](#tier-3-support) hedef üzerinde çalışır.

### Zig libc ile birlikte dağıtılır

Kullanılabilir libc hedeflerini `zig targets` bulabilirsiniz:

```
...
 "libc": [
  "aarch64_be-linux-gnu",
  "aarch64_be-linux-musl",
  "aarch64_be-windows-gnu",
  "aarch64-linux-gnu",
  "aarch64-linux-musl",
  "aarch64-windows-gnu",
  "armeb-linux-gnueabi",
  "armeb-linux-gnueabihf",
  "armeb-linux-musleabi",
  "armeb-linux-musleabihf",
  "armeb-windows-gnu",
  "arm-linux-gnueabi",
  "arm-linux-gnueabihf",
  "arm-linux-musleabi",
  "arm-linux-musleabihf",
  "arm-windows-gnu",
  "i386-linux-gnu",
  "i386-linux-musl",
  "i386-windows-gnu",
  "mips64el-linux-gnuabi64",
  "mips64el-linux-gnuabin32",
  "mips64el-linux-musl",
  "mips64-linux-gnuabi64",
  "mips64-linux-gnuabin32",
  "mips64-linux-musl",
  "mipsel-linux-gnu",
  "mipsel-linux-musl",
  "mips-linux-gnu",
  "mips-linux-musl",
  "powerpc64le-linux-gnu",
  "powerpc64le-linux-musl",
  "powerpc64-linux-gnu",
  "powerpc64-linux-musl",
  "powerpc-linux-gnu",
  "powerpc-linux-musl",
  "riscv64-linux-gnu",
  "riscv64-linux-musl",
  "s390x-linux-gnu",
  "s390x-linux-musl",
  "sparc-linux-gnu",
  "sparcv9-linux-gnu",
  "wasm32-freestanding-musl",
  "x86_64-linux-gnu",
  "x86_64-linux-gnux32",
  "x86_64-linux-musl",
  "x86_64-windows-gnu"
 ],
 ```

Bunun anlamı, bu hedefler için `--library c` *herhangi bir sistem dosyasına bağlı değildir*!
[C merhaba dünya örneğine](#zig-is-also-a-c-compiler) tekrar bakalım:
```
$ zig build-exe hello.c --library c
$ ./hello
Hello world
$ ldd ./hello
	linux-vdso.so.1 (0x00007ffd03dc9000)
	libc.so.6 => /lib/libc.so.6 (0x00007fc4b62be000)
	libm.so.6 => /lib/libm.so.6 (0x00007fc4b5f29000)
	libpthread.so.0 => /lib/libpthread.so.0 (0x00007fc4b5d0a000)
	libdl.so.2 => /lib/libdl.so.2 (0x00007fc4b5b06000)
	librt.so.1 => /lib/librt.so.1 (0x00007fc4b58fe000)
	/lib/ld-linux-x86-64.so.2 => /lib64/ld-linux-x86-64.so.2 (0x00007fc4b6672000)
```

[glibc](https://www.gnu.org/software/libc/), statik olarak inşayı desteklemez, ancak [musl](https://www.musl-libc.org/) destekler:
```
$ zig build-exe hello.c --library c -target x86_64-linux-musl
$ ./hello
Hello world
$ ldd hello
  not a dynamic executable
```

Bu örnekte Zig, musl libc'yi kaynaktan oluşturdu ve ardından ona karşı bağlantı kurdu. x86_64-linux için musl libc derlemesi, [önbellek sistemi](https://ziglang.org/download/0.4.0/release-notes.html#Build-Artifact-Caching) sayesinde kullanılabilir durumda kalır, böylece bu libc'ye tekrar ihtiyaç duyulduğunda anında kullanılabilir olacaktır.

Bu, fonksiyonun herhangi bir platformda mevcut olduğu anlamına gelir. Windows ve macOS kullanıcıları, yukarıda listelenen hedeflerden herhangi biri için Zig ve C kodu oluşturabilir ve libc'ye karşı bağlantı kurabilir. Benzer şekilde kod, diğer mimariler için çapraz derlenebilir:
```
$ zig build-exe hello.c --library c -target aarch64v8-linux-gnu
$ file hello
hello: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 2.0.0, with debug_info, not stripped
```

Bazı açılardan Zig, C derleyicilerinden daha iyi bir C derleyicisidir!

Bu işlevsellik, Zig ile birlikte bir çapraz derleme araç zincirini bir araya getirmekten daha fazlasıdır. Örneğin, Zig'in gönderdiği libc başlıklarının toplam boyutu sıkıştırılmamış 22 MiB'dir. Bu arada, yalnızca x86_64'teki musl libc + linux başlıkları için başlıklar 8 MiB'dir ve glibc için 3.1 MiB'dir (glibc'de linux başlıkları eksik), ancak Zig şu anda 40 libc ile birlikte gönderilmektedir. 444 MiB olacaktı eğer saf bir paketleme kullanılsaydı. Bununla birlikte, bu [process_headers aracı](https://github.com/ziglang/zig/blob/0.4.0/libc/process_headers.zig) ve bazı [eski güzel el emeği sayesinde](https://github.com/ziglang/zig/wiki/Updating-libc), Zig ikili tarball'ları, tüm bu hedefler için libc'yi ve ayrıca derleyici-rt, libunwind ve libcxx'i desteklemesine ve bir clang- olmasına rağmen, toplamda kabaca 30 MiB kalır. uyumlu C derleyicisi. Karşılaştırma için, llvm.org'dan clang 8.0.0'ın Windows ikili yapısı 132 MiB'dir.

Yalnızca [1. Kademe Destek](#tier-1-support) hedeflerinin kapsamlı bir şekilde test edildiğini unutmayın. Daha fazla [libc eklenmesi](https://github.com/ziglang/zig/issues/514) (Windows için dahil) ve [tüm libc'lere karşı derleme için test kapsamı eklenmesi planlanmaktadır](https://github.com/ziglang/zig/issues/2058).

[Zig Paket Yöneticisi olması planlanıyor](https://github.com/ziglang/zig/issues/943) ama henüz yapılmadı. Mümkün olacak şeylerden biri de C kütüphaneleri için bir paket oluşturmak. Bu, [Zig İnşa Sistemini](#zig-build-system) hem Zig programcıları hem de C programcıları için çekici hale getirecektir.

## Zig İnşa Sistemi

Zig bir yapı sistemi ile birlikte gelir, bu nedenle make, cmake veya benzeri bir şeye ihtiyacınız yoktur.
```
$ zig init-exe
Created build.zig
Created src/main.zig

Next, try `zig build --help` or `zig build run`
```

<u>src/main.zig</u>
{{< zigdoctest "assets/zig-code/features/25-all-bases.zig" >}}


<u>build.zig</u>
{{< zigdoctest "assets/zig-code/features/26-build.zig" >}}


Şu `--help` menüsüne bir bakalım.
```
$ zig build --help
Usage: zig build [steps] [options]

Steps:
  install (default)      Copy build artifacts to prefix path
  uninstall              Remove build artifacts from prefix path
  run                    Run the app

General Options:
  --help                 Print this help and exit
  --verbose              Print commands before executing them
  --prefix [path]        Override default install prefix
  --search-prefix [path] Add a path to look for binaries, libraries, headers

Project-Specific Options:
  -Dtarget=[string]      The CPU architecture, OS, and ABI to build for.
  -Drelease-safe=[bool]  optimizations on and safety on
  -Drelease-fast=[bool]  optimizations on and safety off
  -Drelease-small=[bool] size optimizations on and safety off

Advanced Options:
  --build-file [file]         Override path to build.zig
  --cache-dir [path]          Override path to zig cache directory
  --override-lib-dir [arg]    Override path to Zig lib directory
  --verbose-tokenize          Enable compiler debug output for tokenization
  --verbose-ast               Enable compiler debug output for parsing into an AST
  --verbose-link              Enable compiler debug output for linking
  --verbose-ir                Enable compiler debug output for Zig IR
  --verbose-llvm-ir           Enable compiler debug output for LLVM IR
  --verbose-cimport           Enable compiler debug output for C imports
  --verbose-cc                Enable compiler debug output for C compilation
  --verbose-llvm-cpu-features Enable compiler debug output for LLVM CPU features
```

Mevcut adımlardan birinin çalıştırıldığını görebilirsiniz.
```
$ zig build run
All your base are belong to us.
```

İşte bazı örnek derleme betikleri:

- [OpenGL Tetris oyunu için inşa betiği](https://github.com/andrewrk/tetris/blob/master/build.zig)
- [Metal Raspberry Pi 3 arcade oyunu için inşa betiği](https://github.com/andrewrk/clashos/blob/master/build.zig)
- [Kendi-kendine barındırılan Zig derleyicisi için inşa betiği](https://github.com/ziglang/zig/blob/master/build.zig)

## Asenkron Fonksiyonlar aracılığıyla eşzamanlılık

Zig 0.5.0, [asenkron fonksiyonları tanıttı](https://ziglang.org/download/0.5.0/release-notes.html#Async-Functions). Bu özelliğin bir ana bilgisayar işletim sistemine veya hatta yığınla ayrılmış belleğe bağımlılığı yoktur. Bu, bağımsız hedef için asenkron fonksiyonların mevcut olduğu anlamına gelir.

Zig, bir fonksiyonun asenkron olup olmadığını anlar ve asenkron olmayan fonksiyonlarda `async`/`await`'e izin verir; bu, Zig kitaplıklarının, **asenkron G/Ç'ye karşı engelleme konusunda agnostik olduğu anlamına gelir**. [Zig, fonksiyon renklerinden kaçınır](http://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/).

Zig Standart Kütüphanesi, asenkron fonksiyonları M:N eşzamanlılığı için bir iş parçacığı havuzuna çoğaltan bir olay döngüsü uygular. Çok iş parçacıklı güvenlik, aktif araştırma alanlarıdır.

## Desteklenen geniş hedef yelpazesi

Zig, farklı hedefler için destek düzeyini iletmek üzere bir "destek katmanı" sistemi kullanır.

[Zig 0.8.0'dan itibaren Destek Tablosu](/download/0.8.0/release-notes.html#Support-Table)

## Paket geliştiricilerine dostça yaklaşım

Referans Zig derleyicisi henüz tamamen kendi kendine barındırılmıyor, ancak ne olursa olsun, bir sistem C++ derleyicisine sahip olmaktan herhangi bir hedef için tamamen kendi kendine barındırılan bir Zig derleyicisine sahip olmaya gitmek için [tam olarak 3 adım kalacak](https://github.com/ziglang/zig/issues/853). Maya Rashish'in belirttiği gibi, [Zig'i diğer platformlara taşımak eğlenceli ve hızlı](http://coypu.sdf.org/porting-zig.html).

Hata ayıklama olmayan [derleme modları](https://ziglang.org/documentation/master/#Build-Mode) yeniden üretilebilir/deterministiktir.


[İndirme sayfasının bir JSON sürümü var](/download/index.json).

Zig ekibinin birkaç üyesi, paketlerin geliştirilmesi ve bakımı konusunda deneyim sahibidir.

- [Daurnimator](https://github.com/daurnimator) [Arch Linux paketlerine](https://www.archlinux.org/packages/community/x86_64/zig/) bakar.
- [Marc Tiehuis](https://tiehuis.github.io/) Visual Studio Code paketlerine bakar.
- [Andrew Kelley](https://andrewkelley.me/) spent [Debian ve Ubuntu paketlemesine](https://qa.debian.org/developer.php?login=superjoe30%40gmail.com&comaint=yes) tam bir yılını harcadı, ve [nixpkgs](https://github.com/NixOS/nixpkgs/)'e de bakar.
- [Jeff Fowler](https://blog.jfo.click/) Homebrew paketine bakar ve [Sublime paketine](https://github.com/ziglang/sublime-zig-language) başladı (şu anda [emekoi](https://github.com/emekoi) buna bakıyor).
