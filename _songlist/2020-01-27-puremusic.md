---
layout: project
image: /assets/img/song_list/cun.jpg
title: Pure music
sulg: songlist

caption:  电子音乐
date: 27 January 2020
description: >
  心情愉悦 电子音乐

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
            url:'/assets/js/pure_music.json',
            dataType: 'json',
            success: function (result) {
                var ap = new APlayer({
                    element: document.getElementById('aplayer'),
                    lrcType: 3,
                    volume: 0.7,
                    mutex: true,
                    theme: '#32CD32',
                    autoplay: false,
                    order: 'list',
                    listFolded: false,
                    audio: result.Body,
                });
            }
        });
    });
</script> 
