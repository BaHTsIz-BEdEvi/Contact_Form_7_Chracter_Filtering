<p align="justify">In this article, I will explain how to add and edit character filters for an open source WordPress plugin ”Contact Form 7". You can use this method to protect against injection attacks or to filter only characters/words that you do not want. The regulations I will describe apply to the version 5.0.5 of Contact Form 7. I highly recommend that you make the necessary backups before you begin.
The procedures I'm going to tell are about editing the following files;</p>

contact-form-7/includes/formatting.php<br>
contact-form-7/modules/*.php (Değişiklik yapmak istediğiniz girdi bölümü)(e-mail, url vb. alanlar text.php içerisinde yer almaktadır)

<p align="justify">First of all, I would like to tell you that after making the necessary backups, do not hesitate to modify the files of the plugin; Contact Form 7 is a very flexible plugin. The changes I will describe are quite basic and simple changes; The main reason I explain is that those who want to make changes can easily understand the logic of the work by looking at them so that they can make changes according to their wishes.</p>
<p align="justify">We first start filtering through the formatting.php file. This file contains the parameters eleri wpcf7_is_ * _. These parameters allow us to perform the filtering we want. For example, if we take the filtering applied to the Telephone input field;</p>
  
```php
function wpcf7_is_tel( $tel ) {
		$result = preg_match( '/^[+]?[0-9() -]*$/', $tel );
		return apply_filters( 'wpcf7_is_tel', $result, $tel );
	}
 ```

<p align="justify">We can see that it is filtered by a parameter. The parameter name is “wpcf7_is_tel“ and the characters it accepts are numbers between 0-9, "+","(",")" and "-". For example, you want to accept only number characters in the Telephone input field. Then you need to change the parameter as follows;</p>

```php
function wpcf7_is_tel( $tel ) {
	$result = preg_match( '%^[0-9]*$%', $tel );
	return apply_filters( 'wpcf7_is_tel', $result, $tel );
}
 ```

<p align="justify">So far, we've only changed the filtering criteria for the already filtered area, but how we can add a new filtering? No filtering is applied to "Text" and "TextArea" sections. I'll show you the ”Text" section as an example. To do this, you must first create a new filtering parameter or you can use a parameter that already exists.If you like add a new parameter you can add a new parameter to the form “wpcf7_is_ *" in the "formatting.php" file as in the example below;</p>

```php
function wpcf7_is_text( $istext ) {
	$result = preg_match( '%^[A-Za-z]*$%', $istext );
	return apply_filters( 'wpcf7_is_text', $result, $istext );
}
```

<p align="justify">Here we have arranged to accept only the letter characters. According to your request, you can edit it by entering the carousel or range in parentheses. Here "wpcf7_is_text ”has been named of the the parameter; make a note of this parameter name because we need to enter this parameter name for filtering.</p>
<p align="justify">Now we need to add the parameter we created. For this, we will edit the file "contact-form-7/modules/text.php" because our sample has ”Text” sections. In this file we will go to "/* Validation filter */" section. If you examine this section, you can see the filtering sections (such as "wpcf7_is_ * "). However, in the text section, we can see there is only filter "required" section, but not filtering for chracters;</p>

```php
	if ( 'text' == $tag->basetype ) {
		if ( $tag->is_required() && '' == $value ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_required' ) );
		}
	}
```
  
<p align="justify">All we have to do here is to enter an "elseif" command immediately after the "necessary" test section like other sections;</p>
  
```php
	if ( 'text' == $tag->basetype ) {
		if ( $tag->is_required() && '' == $value ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_required' ) );
					} elseif ( '' != $value && ! wpcf7_is_text( $value ) ) {
			$result->invalidate( $tag, wpcf7_get_message( 'invalid_url' ) );
		}
	}
```

<p align="justify">In the "Elseif" command, we need to enter the parameter name that we have previously defined and of course we should pay attention to the parenthesis here. There is also another important point here; How to react if an entry is entered that does not meet the filtering rules. I used the "'invalid_url'" variable in the message section, which is the same as the message that will appear in entries that do not comply with the filing rule in the "url" input sections. If you want, you can either enter another message here or have it react differently instead of sending a message.</p>
<p align="justify">I explained the logic of how to make changes and add-on to those who want to make changes here. You can also make other changes with the same logic; for example, to apply the filtering to the "textarea” section, it is enough to make a change same as “text.php” on "contact-form-7/modules/textarea.php". Those who need help can contact me.</p>
