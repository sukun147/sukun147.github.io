# Catch the Cat


## 一起来抓猫

点击圆点围住猫咪，别让它到达地图边缘！

<script src="/js/catch-the-cat/phaser.min.js"></script>

<script src="/js/catch-the-cat/catch-the-cat.js"></script>

<div align="center"><div id="catch-the-cat"></div></div>
<script>
    window.game = new CatchTheCatGame({
        w: 11,
        h: 11,
        r: 20,
        backgroundColor: 0xeeeeee,
        parent: 'catch-the-cat',
        statusBarAlign: 'center',
        credit: '一起来抓猫！'
    });
</script>

