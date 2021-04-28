---
layout: project
image: /assets/img/song_list/gam.jpg
title: Game music
sulg: songlist

caption: Hydejack as a product page.
date: 1 Jan 2016
description: >
  Using the Free Version of Hydejack as a product page on GitHub Pages.

sitemap: false
comments: true
---

<script type='text/javascript' src='/assets/aplayer/jquery.min.js'></script>

<!-- <script type='text/javascript' src='https://api88.net/api/play/js/?id=4139958112&type=songlist&music=qqmusic&listMaxHeight=500'></script> -->

<script>
$("head").append("<link>");
var css = $("head").children(":last");
css.attr({
    rel: "stylesheet",
    type: "text/css",
    href: "/assets/aplayer/APlayer.min.css"
});
document.write('<div id="aplayer"></div>');
$.getScript('/assets/aplayer/APlayer.min.js', function () {
    $.ajax({
        type: "GET",
        url:'/assets/js/game_music.json',
        dataType: 'json',
        success: function (result) {
            var ap = new APlayer({
                element: document.getElementById('aplayer'),
                lrcType: 3,
                volume: 1,
                mutex: true,
                fixed: false,
                theme: '#32CD32',
                autoplay: false,
                order: 'list',
                listFolded:false,
                audio: result.Body,
            });
        }
    });
});
</script>