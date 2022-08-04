---
layout: project
image: /assets/img/song_list/gam.png
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
<link rel="stylesheet" href="/assets/aplayer/APlayer.min.css">
<div id="aplayer"></div>
<script src="/assets/aplayer/APlayer.min.js"></script>


<!-- <script type='text/javascript' src='https://api88.net/api/play/js/?id=4139958112&type=songlist&music=qqmusic&listMaxHeight=500'></script> -->
<script>
    $(() => {
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