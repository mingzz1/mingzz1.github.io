---
title: "[Wargame.kr] fly to the moon 문제풀이"
date: 2018-10-25
draft: false
category: [wargame]
subcategories: [wargame.kr]
tags: [Write-up, Wargame.kr]
---

다음 문제는 `fly to the moon` 이다.
이 문제는 Javascript 문제이다.  

<!--more-->

문제 정보는 다음과 같다.  

```plain
javascript game.

can you clear with bypass prevent cheating system?
```

문제에 접속하면 정보대로 Javascript로 구현 된 게임이 있다.  

![](/images/wargame.kr/fly_to_the_moon/fly_01.png)

비행기가 하나 있고 이 비행기가 벽을 피해서 날아야 하는 게임이다.
심지어 게임을 좀 해 보면 양쪽 벽이 점점 줄어든다.
벽에 닿으면 죽게 되는데, 그 때 아래와 같은 화면을 확인할 수 있다.  

![](/images/wargame.kr/fly_to_the_moon/fly_02.png)

`31337` 점을 획득하라고 한다.
그런데 아까 말했듯이 조금만 게임해도 벽이 점점 줄어들기 때문에 `31337` 점을 얻는 것은 불가능 해 보인다.
게임을 하면서 `proxy`로 잡아보면, 죽을 때 `high-scores.php`에 `token`과 현재 나의 `score`를 보내는 것을 확인할 수 있다.
그래서 일단 소스코드를 확인 해 보았다.  

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('76 58=["\\89\\66\\68\\68\\154\\68\\67\\92\\59\\71","\\70\\74\\59\\70\\89\\136\\66\\86\\59","\\81\\59\\63\\118\\70\\61\\71\\59","\\124\\66\\64\\70\\118\\70\\61\\71\\59","\\69\\74\\71\\66\\64\\89\\98\\78\\64\\64\\59\\68","\\97\\66\\84\\63\\74\\98\\78\\64\\64\\59\\68","\\61\\104\\148\\59\\70\\63","\\159\\61\\90\\70\\74\\59\\67\\63\\66\\64\\81\\155\\90\\66\\86\\90\\92\\61\\78\\90\\70\\67\\64","\\97\\67\\71\\64","\\61\\86\\86\\69\\59\\63\\136\\59\\86\\63","\\63\\78\\64\\64\\59\\68","\\81\\59\\63\\170\\68\\59\\85\\59\\64\\63\\124\\92\\142\\84","\\63\\61\\77","","\\77\\112","\\70\\69\\69","\\84\\66\\69\\77\\68\\67\\92","\\104\\68\\61\\70\\89","\\59\\67\\70\\74","\\66\\85\\81\\107\\68\\59\\86\\63\\111\\97\\67\\68\\68","\\66\\85\\81\\107\\71\\66\\81\\74\\63\\111\\97\\67\\68\\68","\\91\\74\\66\\81\\74\\111\\69\\70\\61\\71\\59\\69","\\71\\59\\85\\61\\103\\59","\\63\\67\\104\\68\\59","\\64\\61\\64\\59","\\84\\66\\103\\91\\69\\70\\61\\71\\59\\111\\63\\67\\104\\68\\59","\\70\\68\\66\\70\\89","\\63\\59\\112\\63","\\69\\77\\67\\64\\91\\69\\70\\61\\71\\59","\\68\\59\\86\\63","\\66\\85\\81\\91\\69\\74\\66\\77","\\69\\68\\61\\97","\\86\\67\\84\\59\\142\\64","\\104\\67\\70\\89\\81\\71\\61\\78\\64\\84\\149\\77\\61\\69\\66\\63\\66\\61\\64","\\168\\167\\164\\90","\\84\\66\\103\\91\\63\\78\\64\\64\\59\\68","\\71\\67\\64\\84\\61\\85","\\86\\68\\61\\61\\71","\\78\\77\\84\\67\\63\\59\\98\\78\\64\\64\\59\\68\\143\\150","\\86\\67\\84\\59\\151\\78\\63","\\154\\151\\118\\98","\\74\\66\\81\\74\\149\\69\\70\\61\\71\\59\\69\\107\\77\\74\\77","\\63\\61\\89\\59\\64\\146","\\169\\69\\70\\61\\71\\59\\146","\\67\\148\\67\\112","\\74\\63\\85\\68","\\77\\91\\97\\59\\68\\70\\61\\85\\59","\\78\\77\\84\\67\\63\\59\\98\\61\\89\\59\\64\\143\\150","\\63\\74\\112\\155\\90\\152\\74\\71\\66\\69\\63\\66\\67\\64\\90\\145\\61\\64\\63\\61\\92\\67","\\85\\61\\78\\69\\59\\61\\103\\59\\71","\\91\\70\\74\\71\\66\\69\\63\\66\\67\\64","\\85\\61\\78\\69\\59\\61\\78\\63","\\71\\59\\67\\84\\92","\\152\\74\\71\\66\\69\\63\\66\\67\\64\\90\\145\\61\\64\\63\\61\\92\\67","\\77\\67\\81\\59\\175","\\85\\61\\78\\69\\59\\85\\61\\103\\59","\\63\\61\\89\\59\\64\\107\\77\\74\\77","\\81\\59\\63"];62 119(){76 174=73;76 108=93;62 141(){108=173;79 93};62 130(){79 108};73[58[0]]=62(){141();79 93};73[58[1]]=62(){79 130()};76 116=0;62 131(){79 116};62 133(){72(108){116++};79 93};73[58[2]]=62(){79 131()};73[58[3]]=62(){133();79 93};76 123=161;62 138(){123-=20;79 93};62 144(){79 123};73[58[4]]=62(){138();79 93};73[58[5]]=62(){79 144()}};76 95=0;76 110=0;76 134=125;76 75=117;76 94=117;76 101=0;76 102=0;76 99=0;76 96=0;82=106 127(20);88=106 127(20);62 153(){87=106 119();72(58[6]==135 105){105[58[8]](58[7])};110=121[58[11]](58[10])[58[9]];134+=110;65=0;122(65=0;65<20;65++){82[65]=80;88[65]=137};$(58[19])[58[18]](62(83){65=83*25;$(73)[58[15]](58[12],58[13]+65+58[14]);$(73)[58[15]](58[16],58[17])});$(58[20])[58[18]](62(83){65=83*25;$(73)[58[15]](58[12],58[13]+65+58[14]);$(73)[58[15]](58[16],58[17])});$(58[25])[58[26]](62(){$(58[23])[58[22]](58[21]);$(58[25])[58[15]](58[16],58[24]);132();114()})};62 132(){87=106 119();72(58[6]==135 105){105[58[8]](58[7])};75=117;101=0;102=0;99=0;$(58[28])[58[27]](58[13]+0);$(58[30])[58[15]](58[29],75+58[14]);65=0;122(65=0;65<20;65++){82[65]=80;88[65]=137};$(58[30])[58[32]](58[31]);$(58[19])[58[18]](62(83){65=83*25;$(73)[58[15]](58[12],58[13]+65+58[14]);$(73)[58[15]](58[16],58[17])});$(58[20])[58[18]](62(83){65=83*25;$(73)[58[15]](58[12],58[13]+65+58[14]);$(73)[58[15]](58[16],58[17])})};62 114(){95=95+2;72(95>20){95=0};$(58[35])[58[15]](58[33],58[34]+95+58[14]);72(75+32<125){72(75+46<94){75+=4}109{72(75+16<94){75+=2}}};72(75>0){72(75-14>94){75-=4}109{72(75+16>94){75-=2}}};$(58[30])[58[15]](58[29],75+58[14]);102++;72(102>60){102=0;96=126[58[37]](126[58[36]]()*2)};72(82[0]<10){96=1}109{72(88[0]>166){96=0}};65=0;122(65=20;65>0;65--){82[65]=82[65-1];88[65]=88[65-1]};72(96==0){82[0]-=3};72(96==1){82[0]+=3};88[0]=82[0]+87[58[5]]();$(58[19])[58[18]](62(83){$(73)[58[15]](58[29],58[13]+82[83]+58[14])});$(58[20])[58[18]](62(83){$(73)[58[15]](58[29],58[13]+88[83]+58[14])});72(87[58[5]]()>=120){99++;72(99>100){99=0;87[58[4]]();82[0]+=10}};101++;72(101>20){101=0;87.165();$(58[28])[58[27]](58[13]+87[58[2]]())};72(75<=82[18]+20||75+32>=88[18]){87[58[0]]()};72(87[58[1]]()){158(58[38],10)}109{$(58[30])[58[39]](58[31]);$(58[19])[58[15]](58[16],58[24]);$(58[20])[58[15]](58[16],58[24]);$[58[44]]({171:58[40],156:58[41],162:58[42]+115+58[43]+87[58[2]](),157:62(113){147(113)}})}};62 160(){79};62 147(113){$(58[25])[58[45]](113);$(58[25])[58[15]](58[16],58[17])};$(121)[58[52]](62(){$(58[46])[58[15]](58[16],58[17]);140();163(58[47],172);$(58[46])[58[26]](62(){$(58[46])[58[15]](58[16],58[24]);153();114()});$(58[50])[58[49]](62(){$(73)[58[45]](58[48])});$(58[50])[58[51]](62(){$(73)[58[45]](139)})});76 139=58[53];$(121)[58[55]](62(129){94=129[58[54]]-110});76 115=58[13];62 140(){$[58[57]](58[56],62(128){115=128})};',10,176,'||||||||||||||||||||||||||||||||||||||||||||||||||||||||||_0x32bb|x65||x6F|function|x74|x6E|y|x69|x61|x6C|x73|x63|x72|if|this|x68|ship_x|var|x70|x75|return||x67|left_wall|_0x8618x16|x64|x6D|x66|BTunnelGame|right_wall|x6B|x20|x23|x79|true|pos_x|bg_val|t_state|x77|x54|c_w||c_s|c_r|x76|x62|console|new|x2E|_0x8618x3|else|rail_left|x5F|x78|_0x8618x19|updateTunnel|token|_0x8618x6|234|x53|secureGame||document|for|_0x8618x9|x42|500|Math|Array|_0x8618x20|_0x8618x1d|_0x8618x5|_0x8618x7|restartTunnel|_0x8618x8|rail_right|typeof|x4C|400|_0x8618xa|temp|updateToken|_0x8618x4|x49|x28|_0x8618xb|x4D|x3D|showHighScores|x6A|x2D|x29|x4F|x43|initTunnel|x50|x2C|url|success|setTimeout|x44|scoreUpdate|320|data|setInterval|x25|BincScore|470|x30|x35|x26|x45|type|10000|false|_0x8618x2|x58'.split('|'),0,{}))
```

이 코드가 Javascript 코드만 확인 해 본 것이다.
너무 복잡해서 이 상태로는 분석을 못할 것 같아 [JS Beautifier](https://beautifier.io/)를 사용했다.
이 페이지에 소스코드를 넣고 `Beautify JavaScript or HTML` 버튼을 누르면 이쁘게 정리된 코드를 확인할 수 있다.  

```javascript
function secureGame() {
    var _0x8618x2 = this;
    var _0x8618x3 = true;

    function _0x8618x4() {
        _0x8618x3 = false;
        return true
    };

    function _0x8618x5() {
        return _0x8618x3
    };
    this['killPlayer'] = function() {
        _0x8618x4();
        return true
    };
    this['checkLife'] = function() {
        return _0x8618x5()
    };
    var _0x8618x6 = 0;

    function _0x8618x7() {
        return _0x8618x6
    };

    function _0x8618x8() {
        if (_0x8618x3) {
            _0x8618x6++
        };
        return true
    };
    this['getScore'] = function() {
        return _0x8618x7()
    };
    this['BincScore'] = function() {
        _0x8618x8();
        return true
    };
    var _0x8618x9 = 320;

    function _0x8618xa() {
        _0x8618x9 -= 20;
        return true
    };

    function _0x8618xb() {
        return _0x8618x9
    };
    this['shrinkTunnel'] = function() {
        _0x8618xa();
        return true
    };
    this['widthTunnel'] = function() {
        return _0x8618xb()
    }
};
var bg_val = 0;
var rail_left = 0;
var rail_right = 500;
var ship_x = 234;
var pos_x = 234;
var c_s = 0;
var c_r = 0;
var c_w = 0;
var t_state = 0;
left_wall = new Array(20);
right_wall = new Array(20);

function initTunnel() {
    BTunnelGame = new secureGame();
    if ('object' == typeof console) {
        console['warn']('Do cheating, if you can')
    };
    rail_left = document['getElementById']('tunnel')['offsetLeft'];
    rail_right += rail_left;
    y = 0;
    for (y = 0; y < 20; y++) {
        left_wall[y] = 80;
        right_wall[y] = 400
    };
    $('img.left_wall')['each'](function(_0x8618x16) {
        y = _0x8618x16 * 25;
        $(this)['css']('top', '' + y + 'px');
        $(this)['css']('display', 'block')
    });
    $('img.right_wall')['each'](function(_0x8618x16) {
        y = _0x8618x16 * 25;
        $(this)['css']('top', '' + y + 'px');
        $(this)['css']('display', 'block')
    });
    $('div#score_table')['click'](function() {
        $('table')['remove']('#high_scores');
        $('div#score_table')['css']('display', 'none');
        restartTunnel();
        updateTunnel()
    })
};

function restartTunnel() {
    BTunnelGame = new secureGame();
    if ('object' == typeof console) {
        console['warn']('Do cheating, if you can')
    };
    ship_x = 234;
    c_s = 0;
    c_r = 0;
    c_w = 0;
    $('span#score')['text']('' + 0);
    $('img#ship')['css']('left', ship_x + 'px');
    y = 0;
    for (y = 0; y < 20; y++) {
        left_wall[y] = 80;
        right_wall[y] = 400
    };
    $('img#ship')['fadeIn']('slow');
    $('img.left_wall')['each'](function(_0x8618x16) {
        y = _0x8618x16 * 25;
        $(this)['css']('top', '' + y + 'px');
        $(this)['css']('display', 'block')
    });
    $('img.right_wall')['each'](function(_0x8618x16) {
        y = _0x8618x16 * 25;
        $(this)['css']('top', '' + y + 'px');
        $(this)['css']('display', 'block')
    })
};

function updateTunnel() {
    bg_val = bg_val + 2;
    if (bg_val > 20) {
        bg_val = 0
    };
    $('div#tunnel')['css']('background-position', '50% ' + bg_val + 'px');
    if (ship_x + 32 < 500) {
        if (ship_x + 46 < pos_x) {
            ship_x += 4
        } else {
            if (ship_x + 16 < pos_x) {
                ship_x += 2
            }
        }
    };
    if (ship_x > 0) {
        if (ship_x - 14 > pos_x) {
            ship_x -= 4
        } else {
            if (ship_x + 16 > pos_x) {
                ship_x -= 2
            }
        }
    };
    $('img#ship')['css']('left', ship_x + 'px');
    c_r++;
    if (c_r > 60) {
        c_r = 0;
        t_state = Math['floor'](Math['random']() * 2)
    };
    if (left_wall[0] < 10) {
        t_state = 1
    } else {
        if (right_wall[0] > 470) {
            t_state = 0
        }
    };
    y = 0;
    for (y = 20; y > 0; y--) {
        left_wall[y] = left_wall[y - 1];
        right_wall[y] = right_wall[y - 1]
    };
    if (t_state == 0) {
        left_wall[0] -= 3
    };
    if (t_state == 1) {
        left_wall[0] += 3
    };
    right_wall[0] = left_wall[0] + BTunnelGame['widthTunnel']();
    $('img.left_wall')['each'](function(_0x8618x16) {
        $(this)['css']('left', '' + left_wall[_0x8618x16] + 'px')
    });
    $('img.right_wall')['each'](function(_0x8618x16) {
        $(this)['css']('left', '' + right_wall[_0x8618x16] + 'px')
    });
    if (BTunnelGame['widthTunnel']() >= 120) {
        c_w++;
        if (c_w > 100) {
            c_w = 0;
            BTunnelGame['shrinkTunnel']();
            left_wall[0] += 10
        }
    };
    c_s++;
    if (c_s > 20) {
        c_s = 0;
        BTunnelGame.BincScore();
        $('span#score')['text']('' + BTunnelGame['getScore']())
    };
    if (ship_x <= left_wall[18] + 20 || ship_x + 32 >= right_wall[18]) {
        BTunnelGame['killPlayer']()
    };
    if (BTunnelGame['checkLife']()) {
        setTimeout('updateTunnel()', 10)
    } else {
        $('img#ship')['fadeOut']('slow');
        $('img.left_wall')['css']('display', 'none');
        $('img.right_wall')['css']('display', 'none');
        $['ajax']({
            type: 'POST',
            url: 'high-scores.php',
            data: 'token=' + token + '&score=' + BTunnelGame['getScore'](),
            success: function(_0x8618x19) {
                showHighScores(_0x8618x19)
            }
        })
    }
};

function scoreUpdate() {
    return
};

function showHighScores(_0x8618x19) {
    $('div#score_table')['html'](_0x8618x19);
    $('div#score_table')['css']('display', 'block')
};
$(document)['ready'](function() {
    $('p#welcome')['css']('display', 'block');
    updateToken();
    setInterval('updateToken()', 10000);
    $('p#welcome')['click'](function() {
        $('p#welcome')['css']('display', 'none');
        initTunnel();
        updateTunnel()
    });
    $('#christian')['mouseover'](function() {
        $(this)['html']('thx, Christian Montoya')
    });
    $('#christian')['mouseout'](function() {
        $(this)['html'](temp)
    })
});
var temp = 'Christian Montoya';
$(document)['mousemove'](function(_0x8618x1d) {
    pos_x = _0x8618x1d['pageX'] - rail_left
});
var token = '';

function updateToken() {
    $['get']('token.php', function(_0x8618x20) {
        token = _0x8618x20
    })
};
```

소스코드를 살펴보면 `high-scores.php`에 `token`과 `score`를 `POST` 방식으로 전송하는 부분을 찾을 수 있다.  

```javascript
if (BTunnelGame['checkLife']()) {
  setTimeout('updateTunnel()', 10)
} else {
  $('img#ship')['fadeOut']('slow');
  $('img.left_wall')['css']('display', 'none');
  $('img.right_wall')['css']('display', 'none');
  $['ajax']({
    type: 'POST',
    url: 'high-scores.php',
    data: 'token=' + token + '&score=' + BTunnelGame['getScore'](),
    success: function(_0x8618x19) {
      showHighScores(_0x8618x19)
    }
  })
}
```

`data: `부분을 보면 `score`의 값으로 `BTunnelGame['getScore']()` 함수의 결과를 전송 해 준다.  

```javascript
/* ... */

var _0x8618x6 = 0;

/* ... */

function _0x8618x7() {
  return _0x8618x6
};

/* ... */

this['getScore'] = function() {
  return _0x8618x7()
};

/* ... */

function initTunnel() {
    BTunnelGame = new secureGame();
/* ... */
}
```

위의 부분을 살펴보면, `BTunnelGame['getScore']()` 함수는 현재 나의 점수를 의미한다는 것을 알 수 있다.
때문에 만약 이 소스코드를 수정하여 `BTunnelGame['getScore']()` 대신에 `31337`의 값을 넣어준 후, 게임에서 죽으면 점수가 `31337`로 `high-scores.php`에 전송될 것이다.
이를 테스트 해 보기 위해 소스코드를 모두 복사한 후 점수 전송 부분만 아래처럼 바꾸어서 Chrome 개발자도구의 Console에 입력 했다.  

```javascript
if (BTunnelGame['checkLife']()) {
  setTimeout('updateTunnel()', 10)
} else {
  $('img#ship')['fadeOut']('slow');
  $('img.left_wall')['css']('display', 'none');
  $('img.right_wall')['css']('display', 'none');
  $['ajax']({
    type: 'POST',
    url: 'high-scores.php',
    data: 'token=' + token + '&score=' + 31337,
    success: function(_0x8618x19) {
      showHighScores(_0x8618x19)
    }
  })
}
```

![](/images/wargame.kr/fly_to_the_moon/fly_03.png)

위와 같이 `Console` 창에 입력한 후 게임을 플레이 하다가 죽으면 아래와 같이 flag를 얻을 수 있다.  

![](/images/wargame.kr/fly_to_the_moon/fly_04.png)

```plain
FLAG : D566E48378558D07CAFAF1564950482F29F64BAC 
```
