---
layout: project
image: /assets/img/song_list/gam.jpg
title: Game music
sulg: songlist

caption: 游戏音乐
date: 28 January 2020
description: >
    玩游戏时听的音乐

sitemap: false
comments: true
---

<script type='text/javascript' src='/assets/aplayer/jquery.min.js'></script>


<!-- <script type='text/javascript' src='https://api88.net/api/play/js/?id=4139958112&type=songlist&music=qqmusic&listMaxHeight=500'></script> -->
<script>
    $(document).ready(function () {
        if(location.href.indexOf("#reloaded")==-1){
            location.href=location.href+"#reloaded";
            location.reload();
        }
    });

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