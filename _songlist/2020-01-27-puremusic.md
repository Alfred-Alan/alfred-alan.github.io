---
layout: project
image: /assets/img/song_list/cun.jpg
title: Pure music
sulg: songlist

caption: Hydejack as a product page.
date: 1 Jan 2016
description: >
  Using the Free Version of Hydejack as a product page on GitHub Pages.

sitemap: false
comments: true
---
<script type='text/javascript' src='/assets/aplayer/jquery.min.js'></script>

<!-- <script type='text/javascript' src='https://api88.net/api/play/js/?id=3616881472&type=songlist&music=qqmusic'></script> -->

<!-- <iframe src="https://open.spotify.com/embed/playlist/5NnkN8yucyX8nMOK15jAj9" width="300" height="380" frameborder="0" allowtransparency="true" allow="encrypted-media"></iframe> -->

<!-- <script type='text/javascript' src='https://api88.net/api/play/js/?id=3616881472&type=songlist&music=qqmusic&listFolded=false'></script> -->
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
            url: "https://api88.net/api/qqmusic/?key=1193755ae99702b0&id=3616881472&type=songlist&cache=",
            dataType: 'json',
            success: function (result) {
                console.log(result.Body)
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
                    audio: result.Body
                });
            }
        });
    });
</script>