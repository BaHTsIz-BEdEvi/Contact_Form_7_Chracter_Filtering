<p align="justify">Bu yazımda bir açık kaynak WordPress eklentisi olan “Contact Form 7” için karakter filtrelerini ekleme ve düzenleme işlemlerini anlatacağım. Injection saldırılarına karşı koruma sağlamak ya da sadece belirli istemediğiniz karakterleri/kelimeleri filtrelemek için bu yöntemi kullanabilirsiniz. Anlatacağım düzenlemeler Contact Form 7’nin 5.0.5 sürümü için geçerlidir. İşlemlere başlamadan önce gerekli yedeklemeleri yapmanızı tavsiye ederim. Anlatacağım işlemler aşağıdaki dosyaları düzenlemekten ibarettir;</p>

contact-form-7/includes/formatting.php<br>
contact-form-7/modules/*.php (Değişiklik yapmak istediğiniz girdi bölümü)(e-mail, url vb. alanlar text.php içerisinde yer almaktadır)

<p align="justify">Öncelikle söylemek isterim ki gerekli yedeklemeleri yaptıktan sonra eklentinin dosyaları üzerinde değişiklik yapmaktan çekinmeyin; Contact Form 7 oldukça esnek bir eklenti. Anlatacağım değişiklikler oldukça temel ve basit değişiklikler; anlatmamın temel sebebi değişiklik yapmak isteyenlerin bunlara bakarak kolayca eklentinin çalışma mantığını anlayabilmesi böylece kendi isteklerine göre değişiklikleri yapabilmesi.</p>
<p align="justify">Filtrelemelere öncelikle  “formatting.php” dosyasından başlıyoruz. Bu dosyada “wpcf7_is_*“ parametreleri tutuluyor. Bu parametreler istediğimiz filtrelemeleri yapmamızı sağlıyor. Örneğin “Telefon” girdi alanına uygulanan filtrelemeyi ele alırsak;</p>
  
```php
function wpcf7_is_tel( $tel ) {
		$result = preg_match( '/^[+]?[0-9() -]*$/', $tel );
		return apply_filters( 'wpcf7_is_tel', $result, $tel );
	}
 ```

<p align="justify">Şeklinde bir parametre ile filtrelendiğini görebiliyoruz. Parametre ismi “wpcf7_is_tel” ve kabul ettiği karakterler ise 0-9 arası rakamlar, “+”,“(“, “)” ve “-“ karakterleri. Örneğin “Telefon” girdi alanına sadece sayı karakterlerini kabul etmek istiyorsunuz. O halde parametreyi aşağıdaki şekilde değiştirmeniz gerekli;</p>

```php
function wpcf7_is_tel( $tel ) {
	$result = preg_match( '%^[0-9]*$%', $tel );
	return apply_filters( 'wpcf7_is_tel', $result, $tel );
}
 ```

<p align="justify">Buraya kadar sadece hali hazırda filtreleme yapılan alanın filtreleme kriterini değiştirdik, peki yeni bir filtreleme nasıl ekleyebiliriz. “Text” ve “TextArea” bölümlerine herhangi bir filtreleme uygulanmamaktadır. Ben örnek olarak “Text” bölümünü göstereceğim. Bunun için öncelikle yeni bir filtreleme parametresi oluşturmanız gerekmektedir ya da hali hazırda olan bir parametreyi de kullanabilirsiniz tabii ki. Yeni bir parametreyi aşağıdaki örnekteki gibi “formatting.php” dosyasının içerisinde “wpcf7_is_*“ parametrelerinin olduğu yere ekleyebilirsiniz;</p>

```php
function wpcf7_is_text( $istext ) {
	$result = preg_match( '%^[A-Za-z]*$%', $istext );
	return apply_filters( 'wpcf7_is_text', $result, $istext );
}
```

<p align="justify">Burada sadece harf karakterlerini kabul etmesi için düzenlemiş olduk. İsteğinize göre parantez içine karater yada aralık girerek düzenleyebilirsiniz. “wpcf7_is_text” parametrenin ismi olmuş oldu burada; bu parametre ismini bir kenara not alın çünkü filtreleme için bu parametre ismini girmemiz gerekiyor.</p>
<p align="justify">Şimdi ise oluşturduğumuz parametreyi eklememiz gerekiyor. Bunun için ise örneğimiz “Text” bölümleri olduğu için “contact-form-7/modules/text.php” dosyasını düzenleyeceğiz. Bu dosya içerisinde ”/* Validation filter */” bölümüne gideceğiz. Burayı incelerseniz “Url” bölümleri gibi bölümlere filtreleme uygulandığı (filtreleme yapılan bölümleri “wpcf7_is_*” parametrelerinden anlayabiliriz) görebiliriz. Fakat “text” bölümünde sadece “gerekli” bölüm olup olmadığını test ettiğini fakat filtreleme yapmadığını görebiliriz;</p>

```php
	if ( 'text' == $tag->basetype ) {
		if ( $tag->is_required() && '' == $value ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_required' ) );
		}
	}
```
  
<p align="justify">Burada yapmamız gereken tek şey diğer bölümlerde olduğu gibi “gerekli” bölüm olup olmadığını anlamak için yapılan testten hemen sonraya bir “elseif” komutu girmek olacak;</p>
  
```php
	if ( 'text' == $tag->basetype ) {
		if ( $tag->is_required() && '' == $value ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_required' ) );
					} elseif ( '' != $value && ! wpcf7_is_text( $value ) ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_url' ) );
		}
	}
```

<p align="justify">“Elseif” komutu içerisine daha önce belirlemiş olduğumuz parametre ismini girmemiz gerekecek ve tabii ki burada parantezlere dikkat etmekte yarar var. Burada ayrıca önemli bir nokta daha var; Eğer filtreleme kurallarına uymayan bir girdi girilirse nasıl bir tepki vereceği. Ben burada çıkarılacak mesaj kısmına “'invalid_url'” değişkenini kullandım ki bu değişken “url” girdi bölümlerinde filteleme kuralına uymayan girdilerde çıkacak mesajla aynı. İsterseniz buraya başka bir mesaj da girebilir ya da mesaj çıkartmak yerine farklı bir tepki vermesini de sağlayabilirsiniz.</p>
<p align="justify">Burada değişiklik yapmak isteyenlere temel olarak değişiklikleri nasıl yapacağınızı ve eklentini mantığını anlattım. Diğer değişiklikleri de aynı mantıkla yapabilirsiniz; örneğin benzer olarak “textarea” bölümüne filtreleme uygulamak için “text.php” üzerinde yaptığımız değişikliği “contact-form-7/modules/textarea.php” üzerinde yapmamız yeterli olacaktır. Yardıma ihtiyacı olanlar benimle iletişime geçebilir.</p>
