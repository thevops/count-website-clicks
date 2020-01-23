# count-website-clicks
Zliczanie liczby kliknięć w linki na stronie wraz z innymi informacjami



# Poniżej przedstawiam 2 sposoby

### Założenia
- zliczanie kliknięć dla: `<a href="https://strona.pl">strona.pl</a>`


## 1. Z użyciem parametru `id` oraz wywołań AJAX

### Wymagania
- jquery

### Kroki
1. Dodać parametr `id` do znacznika `<a></a>` -> `<a href="https://strona.pl" id="moj_link_1">strona.pl</a>`
2. Dodać skrypt:
```
<script type="text/javascript">
$("#moj_link_1").on('click', function(){
    $.ajax({
        type: "POST",
        url: "counter.php",
        data: "moj_link_1",
    })
});
</script>
```
3. Dodać plik `counter.php` z zwartością:
```
<?php

function get_country_code($ip)
{
  $cmd = "curl -m 2 -s https://ipinfo.io/" . $ip . "/country" . " || echo NULL";
  $out = shell_exec($cmd);
  $out = str_replace("\n", "", $out);
  return $out;
}

if(isset($_POST['moj_link_1']))
{
  $CN = get_country_code($_SERVER['REMOTE_ADDR']);
  $datetime = gmdate("Y-m-d H:i:s", $_SERVER['REQUEST_TIME'] + 3600);
  $text = $datetime . ' ' . $_SERVER['REMOTE_ADDR'] . ' ' . $CN . ' ' . $_SERVER['HTTP_USER_AGENT'];
  file_put_contents('moj_link_1.log', $text . "\n", FILE_APPEND | LOCK_EX);
  exit();
}

?>
```
Skrypt zapisuje date, godzinę, adres IP, kraj oraz User-Agent klienta w pliku `moj_link_1.log`



## 2. Z użyciem skryptu przekierowującego

### Kroki
1. Dodać skrypt `redirect.php`:
```
<?php

function get_country_code($ip)
{
  $cmd = "curl -m 2 -s https://ipinfo.io/" . $ip . "/country" . " || echo NULL";
  $out = shell_exec($cmd);
  $out = str_replace("\n", "", $out);
  return $out;
}


if(isset($_GET['url']))
{
	// log request for counting
	$destination = $_GET['url'];
	$CN = get_country_code($_SERVER['REMOTE_ADDR']);
	$datetime = gmdate("Y-m-d H:i:s", $_SERVER['REQUEST_TIME'] + 3600);
	$text = $datetime . ' ' . $_SERVER['REMOTE_ADDR'] . ' ' . $CN . ' ' . $destination . ' ' . $_SERVER['HTTP_USER_AGENT'];
	file_put_contents('count_clicks.log', $text . "\n", FILE_APPEND | LOCK_EX);

	// redirect
	header("Location: ". $destination);
}

?>
```
Skrypt zapisuje date, godzinę, adres IP, kraj, kliknięty link oraz User-Agent klienta w pliku `count_clicks.log`

2. Zmienić adres URL na `<a href="redirect.php?url=https://strona.pl" id="moj_link_1">strona.pl</a>`
