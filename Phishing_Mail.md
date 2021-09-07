# Phishing Mail
Bugün karşılaştığımız bir "Phishing Saldırısından" sizlere bahsetmek istiyorum. Kullanıcılarımızın bir kısmına iletilen bir mailde saldırgan tarafından varlığı bilinen bir adresin parolasının kısa bir süre içerisinde sona ereceği ve mail adresinin aktif olan parolası ile giriş yapılması gerektiği belirtilmiş. 

![ss_1](https://user-images.githubusercontent.com/68084571/132403642-e14fb772-af58-4b4c-bd84-a44a8a61240e.png)

Link kısmında bulunan Web adresi ise Türkiye' de bulunan bir domaine ait olup, sayfanın geri döndüğü cevap kontrol edildiğinde ise URI kısmında bulunan base64 ifadesinde bulunan sayfaya yönlendirme yapmaya yaramaktadır.

![ss_14](https://user-images.githubusercontent.com/68084571/132403715-477c0c18-31f1-4940-8efe-c8a1b3733570.png)

![ss_9](https://user-images.githubusercontent.com/68084571/132403751-c8c867ab-31a9-41e9-8a75-f334801ddf38.png)

Yönlendirilen adres standart bir Outlook mail girişi yapılan bir siteye benzese de URL kısmından aslında oltalama saldırısı olduğu anlaşılmaktadır. 

![ss_7](https://user-images.githubusercontent.com/68084571/132403789-2bb203dc-f5d2-412b-a991-79a4174b47de.png)

Ancak alınmaya çalışan parola bilgilerinin hangi adreste nasıl saklandığını bulmaya çalışmak amacıyla elde ettiğim bilgileri bu aşamada bırakmayıp bir seviye daha ileri götürmek istedim ve sayfa kaynağını incelediğim anda URL Encoded bir Javascript kodu ile karşılaştım.

![ss_2](https://user-images.githubusercontent.com/68084571/132403841-ae27147d-c90d-44d3-b669-f6c4a87a54aa.png)

Unescape fonksiyonunun içerisinde bulunan ifadeyi URL Decode ile okunabilir bir formata getirdiğimde karşılaştığım HTML dökümanı sayfanın standart bir Microsoft Login Portal' ı gibi görünmesini sağlamak için yazıldığını anladım. Burada kod içerisinden bazı noktalara değinecek olursam, Microsoft logoları ve giriş yapıldıktan sonra kullanıcının redirect edileceği adresler haricinde kod içerisinde çok özen gösterilmemiş noktalar bulunmaktadır.

![ss_10](https://user-images.githubusercontent.com/68084571/132403945-1c600979-0257-4fe4-b3fb-df3f2f78513c.png)

![ss_11](https://user-images.githubusercontent.com/68084571/132403954-d24a0427-a64c-43a5-b0f8-50abf1ce24ca.png)

Kod içerisinde bulunan ve asıl olarak ilgimi çeken nokta dışarıda bulunan bir domain' e "POST" request ile alınan verilerin gönderilmesi oldu.

![ss_3](https://user-images.githubusercontent.com/68084571/132403960-164fc24d-6d99-4bd0-b7cb-48b425347dd4.png)

Bunun üzerine elimde olan adresi önce "virustotal.com" daha sonrasında "Shodan" ve manuel olarak inceledim. Ancak virustotal tarafında herhangi bir zararlı aktivite ibaresi bulunmamaktaydı. Shodan ise web sitesinin Romanya üzerinde bulunduğunu belirtti.

![ss_12](https://user-images.githubusercontent.com/68084571/132404043-ec7f3a05-4be8-4c89-91e1-71a52728d8bb.png)

Asıl bulguları manuel olarak inceleme sırasında elde ettim. HTML kodu üzerinde bulunan uzantıyı ziyaret ettiğimde inceleyebileceğim herhangi bir `php`  kodu ile karşılaşmadım ve bu nedenle server-side bir işlemin olduğunu varsaydım. Site içinde bulunan diğer dosyaları gezdiğimde ise karşıma dünyanın çeşitli ülkelerinden (Türkiye de dahil olmak üzere) birçok kişiden çalınmış olan parolaların bulunduğu bir dosya ile karşılaştım. 

![ss_13](https://user-images.githubusercontent.com/68084571/132404077-f33ae157-5355-4a18-920a-31e02f13c990.png)

![ss_4](https://user-images.githubusercontent.com/68084571/132404186-bef861c1-1cb2-40b7-ba3d-b560db7229c0.png)

![ss_5](https://user-images.githubusercontent.com/68084571/132404114-5fe41335-5083-43b8-a0fc-9a5bd7041442.png)

Dosya içerisinde bilinen şirketlerin çalışanlarının da bilgileri bulunmakta. Bu durumla ilgili öncelikle `abuse@amazonaws.com` ile iletişime geçtik, bunun sebebi de login sayfası aslında AWS serverlarında yayınlanmaktaydı.

![ss_6](https://user-images.githubusercontent.com/68084571/132404264-3bd166e6-9bf3-4af3-b7d6-bf4893e2a10d.png)

Gelen hızlı dönüş sonrası kontrolünü sağladığıma göre sayfa artık **erişilemez** durumda. PHP dosyasının bulunduğu siteyi de aynı şekilde şikayet edeceğiz. Ancak o zamana kadar aynı şekilde Türkiye içerisinde bulunan şirketlere ulaşıp durumdan haberdar etmek istiyoruz.

*Disclaimer*
*	Ben Siber Güvenlik dünyasının savunma kısmında çalışan ya da çalışmayı düşünen bir kişi değilim (en azından yakın zamanda), daha çok saldırı tarafı ile ilgileniyorum. Bu nedenle yukarıda bulunan aşamalarda çok büyük ihtimalle daha detaylı ve işin kaynağına giden noktalar bulunabilir. Eğer daha detaylı inceleme yolları varsa ve **paylaşmak** isterseniz benimle iletişime geçerseniz sevinirim.
