Yapay Zeka Kullanımı : 

Bu dosya, Kristal Madenleri projesinin geliştirilmesi sırasında yapay zeka araçlarına sorulan soruları cevaplarını içerir.

Kullanılan Araç: Claude 

Soru:
requestAnimationFrame ile setInterval arasındaki fark nedir?
Cevap:
requestAnimationFrame (rAF), tarayıcının bir sonraki çizim karesinden hemen önce belirtilen fonksiyonu çağırır. setInterval'dan temel farkları şunlardır:
•	Sekme arka plandayken durur: Kullanıcı başka bir sekmeye geçtiğinde rAF çağrısı askıya alınır; bu sayede boşa CPU tüketilmez.
•	Ekran yenileme hızına senkronize olur: 60Hz ekranda saniyede ~60 kez, 144Hz ekranda ~144 kez çağrılır. setInterval ise sabit bir milisaniye değeriyle çalışır ve ekran yenileme hızıyla senkron değildir.
•	Delta time zorunludur: rAF farklı ekranlarda farklı hızlarda çalışacağı için nesne hareketleri her zaman dt (geçen süre) ile çarpılmalıdır; aksi takdirde oyun 144Hz ekranda 60Hz'e göre 2.4 kat hızlı çalışır.
Temel döngü yapısı:
let lastTime = 0;

function gameLoop(timestamp) {
    const dt = (timestamp - lastTime) / 1000; // saniye cinsinden
    lastTime = timestamp;

    update(dt);  // fizik, giriş, mantık
    draw();      // canvas çizimi

    requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
dt değerini 0.05 gibi bir üst sınırla kesmek iyi bir pratiktir; böylece sekme arka plandan öne geldiğinde birikmüş büyük dt değeri nesneleri duvarların içinden geçirmez.

Soru: 
javascript kullanarak ve hiçbir kütüphane kullanmadan ses nasıl üretebilirim?
Cevap:
JavaScript'te ses üretmek için Web Audio API kullanılır — hiçbir kütüphane gerekmez, tarayıcıya yerleşik gelir.
Temel Kavramlar
Web Audio API, bir ses grafiği üzerine kuruludur:
AudioContext → OscillatorNode → GainNode → destination (hoparlör)
Önemli Noktalar
Kavram	Açıklama
AudioContext	Tüm ses işlemlerinin merkezi
OscillatorNode	Ham ses dalgası üretir
GainNode	Ses seviyesini (0.0 – 1.0) kontrol eder
destination	Hoparlör çıkışı
Dalga tipleri:
•	sine → yumuşak, saf ton
•	square → keskin, elektronik ses
•	sawtooth → keskin, tiz ses
•	triangle → sine'ye benzer ama daha sert
Fade In / Fade Out (Tık Sesi Olmadan)
const gain = ctx.createGain();
gain.gain.setValueAtTime(0, ctx.currentTime);
gain.gain.linearRampToValueAtTime(0.5, ctx.currentTime + 0.01); // fade in
gain.gain.linearRampToValueAtTime(0, ctx.currentTime + 0.99);   // fade out
Not: Tarayıcılar AudioContext'i kullanıcı etkileşimi olmadan başlatmayı engeller. Bu yüzden her zaman bir butona bağlı fonksiyon içinde çağır.
İnteraktif bir synthesizer demosu ister misin?

Soru:
Web Audio API ile oyun sesi nasıl yapılır? Harici ses dosyası kullanmak istemiyorum.
Cevap:
Web Audio API, tarayıcı içinde osilatörlerle ses üretmenizi sağlar. Temel yapı:
const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
Dikkat edilmesi gereken iki nokta:
1.	Kullanıcı etkileşimi zorunlu: Tarayıcılar güvenlik politikası gereği AudioContext'in bir kullanıcı etkileşimi (tıklama, tuş basma) olmadan başlatılmasına izin vermez. Oyun başlatma butonuna tıklandığında audioCtx oluşturulmalıdır.
2.	Her ses için yeni osilatör: Osilatörler tek kullanımlıktır; start() ve stop() çağrıldıktan sonra tekrar kullanılamaz. Her ses çalınacağında yeni bir createOscillator() çağrısı yapılmalıdır.
Arkaplan müziği için setInterval ile notalar sırayla çalınabilir:
const melody = [220, 246, 261, 293, 329, 293, 261, 246];
let noteIndex = 0;
setInterval(() => {
    playNote(melody[noteIndex % melody.length]);
    noteIndex++;
}, 220); // her 220ms'de bir nota
________________________________________

Kullanılan Araç: Gemini 
1. Soru: Maden kazma mekaniğinde oyuncunun çok uzaktaki blokları kırmasını nasıl engellerim?
AI Cevabı: Öklid mesafesi (Euclidean distance) formülünü kullanabilirsin: $\sqrt{(x_2-x_1)^2 + (y_2-y_1)^2}$. Eğer bu değer belirlediğin menzilden büyükse kazma işlemini iptal et.

2.Soru: Dışarıdan dosya yüklemeden ses efekti nasıl ekleyebilirim?
AI Cevabı: Web Audio API kullanarak "Oscillator" düğümleri oluşturabilirsin. Kare veya testere dişli dalga formları 8-bit retro sesleri simüle etmek için idealdir.

3. Soru: Blokları kırdığımda ortaya çıkan toz ve kıvılcım efektlerini (particle) nasıl yaparım?
AI Cevabı: Küçük kare veya dairelerden oluşan bir dizi (array) oluştur. Her parçacığa rastgele bir başlangıç hızı ve ömür (life) ver. Her karede ömürlerini azalt ve bittiğinde diziden sil.

4.Soru: Oyunumdaki karakterlerimi asset kullanmadan yapmak istiyorum bunun için nasıl bir yol izlemeliyim?
AI Cevabı:
Oyun varlıklarını (asset) dışarıdan hazır almadan veya profesyonel çizim araçlarına girmeden, tamamen kod (Canvas API) veya temel geometrik formlarla oluşturmak, projenize hem benzersiz bir tarz katar hem de dosya boyutunu inanılmaz düşürür.
Asset kullanmadan karakter tasarlamak için izleyebileceğiniz yollar şunlardır:
1. Prosedürel Geometrik Çizim 
Karakterinizi kareler (fillRect), daireler (arc) ve çizgilerden (lineTo) oluşan bir kompozisyon olarak düşünün.
•	Katmanlama: Önce gövdeyi, sonra kafayı, en son detayları (gözler, kask lambası) çizin. ctx.save() ve ctx.restore() işlemleri, karakterin yönünü değiştirirken (aynalama) işinizi çok kolaylaştırır.
•	Fonksiyonlaştırma: Karakteri tek bir fonksiyon içinde, parametre alacak şekilde (drawPlayer(x, y, color, state)) kurgulayın.
2. Parçalı Animasyon (Procedural Animation)
Asset kullanmadığınızda karakterin canlı görünmesi için "parçaların" hareket etmesi gerekir.
•	Salınım (Oscillation): Karakterin yürüdüğünü hissettirmek için bacakların yüksekliğini Math.sin(zamana_bağlı_değişken) ile değiştirin. 
•	Parçacık Efektleri: Karakter hareket ederken arkasından küçük toz kareleri (particles) çıkartarak hareket hissini güçlendirebilirsiniz.
3. Pixel Art'ı Kodla Modelleme (Grid Method)
Eğer belirli bir karakter tasarımı istiyorsanız, bunu bir dizi (array) içine 0 ve 1'lerden oluşan bir matris olarak tanımlayabilirsiniz:
JavaScript
const playerModel = [
  [0, 1, 1, 0],
  [1, 1, 1, 1],
  [0, 1, 1, 0],
  [0, 1, 1, 0]
];
// Bu matrisi iç içe iki döngüyle, her '1' için küçük bir kare çizerek ekrana basın.
4. Işık ve Gölgelendirme (Gradients)
Düz renkler karakteri "yassı" gösterir. Canvas'ın createRadialGradient özelliğini kullanarak karakterinize derinlik katabilirsiniz.
5. Durum Bazlı Görseller (State Machines)
Asset kullanmadığınız için karakterin o anki durumuna göre çizim kurallarını değiştirin:
•	Zıplarken: Gövdeyi biraz dikeyde uzatın (scale).
•	Kazarken: Kazma çizgisini (lineTo) ileri geri hareket ettirin.
•	Düşerken: Karakterin saç veya kask detaylarını yukarı doğru kaydırın.
Özetle Stratejiniz:
Asset yüklemek yerine, karakteri "çizen bir algoritma" yazıyorsunuz. Bu yöntemle karakterin rengini, boyutunu ve hatta ruh halini sadece bir değişken değiştirerek saniyeler içinde güncelleyebilirsiniz. Bu, özellikle "Kristal Madenleri" gibi teknik odaklı projelerde geliştiriciye büyük bir esneklik sağlar.

5. Soru: Karakterim hareket ederken arkasında neden iz bırakıyor? 
AI Cevabı: Canvas bir resim defteri gibidir; üzerine yeni bir şey çizdiğinde eskisi silinmez. Karakterin yeni konumunu çizmeden önce clearRect ile ekranı temizlemeli veya her karede arka planı yeniden boyayarak eski çizimleri örtmelisin.
6. Soru: Karakterimi assetler kullanmadan oluşturmak istiyorum bunu nasıl ypabilirim?
AI  cevabı : 
Karakterini dışarıdan hazır bir resim (asset) yüklemeden oluşturmak, aslında Canvas API'nin çizim yeteneklerini kullanarak kodla bir "robot" inşa etmek gibidir. Senin projen özelinde, karakterini pikselleri ve geometrik şekilleri üst üste dizerek nasıl oluşturabileceğini adım adım anlatıyorum.
1. Temel Geometrik Parçalarla İnşa (Layering)
Karakterini tek bir parça olarak değil, birleştirilebilir parçalar olarak hayal et. HTML5 Canvas'ta her parça için farklı bir renk (fillStyle) ve konum (fillRect) belirlersin.
•	Gövde: Genelde bir dikdörtgen.
•	Kafa: Daha küçük bir kare veya bir daire (arc).
•	Aksesuarlar: Kask lambası, kazma veya çanta gibi detayları eklemek için çok küçük kareler kullanabilirsin.
2. Koordinat Sistemi ve "Anchor Point" (Çıpa Noktası)
Karakterini çizerken en önemli kural, tüm parçaları karakterin ana koordinatlarına (player.x, player.y) göre hesaplamaktır. Eğer parçaları sabit sayılarla (örneğin x=100, y=100) çizersen, karakter hareket ettiğinde kafası arkada kalır.
Örnek Mantık:
JavaScript
// Karakterin gövdesi (Ana gövde)
ctx.fillRect(player.x, player.y, player.w, player.h);

// Kafa (Gövdenin üstünde ve ortalanmış)
ctx.fillRect(player.x + 4, player.y - 12, 16, 12);
3. Pikselleri "Aynalama" (Facing Direction)
Resim kullanmadığın için karakterin sağa mı sola mı baktığını anlamak için tüm çizimi baştan yazmana gerek yok. Canvas'ın scale özelliğini kullanarak çizimi olduğu gibi ters çevirebilirsin:
JavaScript
ctx.save(); // Mevcut durumu kaydet
if (!player.facingRight) {
    ctx.translate(sx + player.w, sy); // Çizim fırçasını karakterin sonuna taşı
    ctx.scale(-1, 1); // Yatayda -1 katına çıkar (aynada yansıt)
}
// Çizim kodları burada yer alır...
ctx.restore(); // Her şeyi normale döndür
4. Animasyon İllüzyonu (Procedural Animation)
Asset kullanmadan hareket hissi yaratmanın en iyi yolu matematiksel fonksiyonlardır. Örneğin yürüme sırasında bacakların hareket etmesi için Math.sin() (sinüs dalgası) kullanabilirsin. Sinüs dalgası -1 ile 1 arasında gidip geldiği için bacak piksellerini aşağı-yukarı oynatır.
5. Işık ve Gölge Efektleri
Karakterin "düz" durmaması için kodla ışık hüzmesi ekleyebilirsin. Örneğin senin madenci karakterinde kask lambası için createRadialGradient kullanarak etrafa yayılan bir ışık halkası oluşturabilirsin. Bu, karakterin asset kullanmamasına rağmen profesyonel görünmesini sağlar.
Neden Bu Yöntem Daha İyi?
1.	Sınırsız Kişiselleştirme: Bir değişkeni değiştirerek karakterin rengini anında değiştirebilirsin (örneğin canı azaldığında kırmızıya dönmesi).
2.	Sıfır Yükleme Süresi: Resim dosyalarının yüklenmesini beklemen gerekmez, kod çalıştığı anda karakter oradadır.
3.	Hafiflik: Projenin dosya boyutu KB seviyesinde kalır.
Kısacası; asset kullanmadan karakter yapmak, her bir karede (frame) bilgisayara "Şu koordinata şu renk kareyi koy" emrini çok hızlı bir şekilde vermektir.


Not: Proje kodları, yapay zeka tarafından verilen cevaplar referans alınarak grup üyeleri tarafından yazılmıştır. Yapay zekadan alınan cevaplar birebir kopyalanmamış olup rehber olması amacıyla kullanılmıştır.


