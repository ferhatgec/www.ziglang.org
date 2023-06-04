---
title: "Kod Örnekleri"
mobile_menu_title: "Kod Parçaları"
toc: true
---

## Bellek taşması tespiti
`std.heap.GeneralPurposeAllocator`'ı kullanarak çifte boşa bırakma ve bellek taşması gibi sorunları tespit edebilirsiniz.

{{< zigdoctest "assets/zig-code/samples/1-memory-leak.zig" >}}


## C ile birliktelik
C başlık dosyasını dahil etme ve hem linc hem raylib'i bağlama örneği.

{{< zigdoctest "assets/zig-code/samples/2-c-interop.zig" >}}


## Zigg Zagg
Zig kodlama görüşmeleri için *optimize* edilmiştir (aslında öyle değil).

{{< zigdoctest "assets/zig-code/samples/3-ziggzagg.zig" >}}

## Yaygın Türler
Zig'te türler comptime değerleridir ve yayın algoritmaları ve veri yapılarını yazmak için tür döndüren fonksiyonları kullanırız.
Bu örnekte, basit yaygın sıra yapısı yazıyoruz ve davranışlarını test ediyoruz .

{{< zigdoctest "assets/zig-code/samples/4-generic-type.zig" >}}


## Zig'te cURL'ü kullanmak

{{< zigdoctest "assets/zig-code/samples/5-curl.zig" >}}
