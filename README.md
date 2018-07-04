# coloriage

### logic jeu coloriage

#### coloration d'un dessin au trait en SVG > export dans un canvas avec fond personnalisé

````js

(function(m){
    var _m = m,
    svgcnt,
    _carac = '',
    curColor,
    scontext,
    scanvas,
    url = '',
    divzoom,
    curScale = 1,
    xs = 0, ys = 0,             // origine x,y mousedown pour drag
    oldxs = 0, oldys = 0,
    xd = 0, yd = 0,             // range drag
    xod = 0, yod = 0,           // offsetX, offsetY
    upmouse = false,
    rowcolor,
    nx = 0, ny = 0,
    oldx = 0, oldy = 0,
    tx = 0,
    ty = 0,
    decalTop = 0,


    transendZoom = function(e,c,t){
        // console.log('end trnsition');
        _m.removeAclass('contsvg','forZoom');
        //_m.$dc('contsvg').style.top = _m.$dc('contsvg').getBoundingClientRect().top+"px";
    },
    
    transendZoom2 = function(e,c,t){
        // console.log('end trnsition2');
        _m.removeAclass('contsvg','forZoom');
        //_m.$dc('contsvg').style.top = _m.$dc('contsvg').getBoundingClientRect().top+"px";
    },

    getTranslateX = function (myElement) {
        var style = window.getComputedStyle(myElement);
        try{
            var matrix = new WebKitCSSMatrix(style.webkitTransform);
        }catch(e){
            try{
                var matrix = new MSCSSMatrix(style.msTransform);
            }catch(e){
                return 0;
            }
        }
        // console.log('translateX : ', matrix.m41);
        return matrix.m41;
    }

    getTranslateY = function (myElement) {
        var style = window.getComputedStyle(myElement);
        try{
            var matrix = new WebKitCSSMatrix(style.webkitTransform);
        }catch(e){
            try{
                var matrix = new MSCSSMatrix(style.msTransform);
            }catch(e){
                return 0;
            }
        }
        // console.log('translateY : ', matrix.m42);
        return matrix.m42;
    }

    mouseDown = function(e,c,t){
        // console.log('mouseDown');
        tx = getTranslateX(_m.$dc('contsvg'));
        ty = getTranslateY(_m.$dc('contsvg'));
        var id = e.currentTarget.getAttribute('id');

        _m.$dc('contsvg').style.msTransform = "translate(" + (tx) + "px, " + ty + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.WebkitTransform = "translate(" + (tx) + "px, " + ty + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.transform = "translate(" + (tx) + "px, " + ty + "px) scale(" + curScale + ")";
        
        var ysvg = _m.$dc('contsvg').getBoundingClientRect().top;
        var xsvg = _m.$dc('contsvg').getBoundingClientRect().left;
        var xz = _m.$dc('zoom').getBoundingClientRect().left;
        _m.addAclass('contsvg','forZoom');

        // no touch
        if(e.changedTouches === undefined || !e.changedTouches === false){
            xs = e.x === undefined ? e.pageX : e.x;
            ys = e.y === undefined ? e.pageY : e.y;
            if(oldxs != xs && oldys != ys) _m.listenerAdd(id, 'mousemove', mouseDrag, true);
        }    
        //touch
        if(e.changedTouches && e.touches.length == 1) {
            xs = e.changedTouches[0].clientX;
            ys = e.changedTouches[0].clientY;
            if(oldxs != xs && oldys != ys) _m.listenerAdd(id, 'touchmove', mouseDrag, true);
        }
       
        oldx = xs, oldy = ys;

        if(oldxs == xs && oldys == ys) return false;
        
        if(curScale <=1){
            ys -= ysvg;
            xs -= (xsvg-xz);
        }
        
        oldxs = xs;
        oldys = ys;
        
        upmouse=false;
    },
    
    mouseUp = function(e,c,t){
        // console.log('mouseUp');
        // _m.removeAclass('contsvg','forZoom');
        var id = e.currentTarget.getAttribute('id');
        if(e.changedTouches === undefined) {
            _m.listenerRemove(id, 'mousemove', true);
            // _m.forceRemove(id, 'mousemove', mouseDrag, true);
        }else{
            _m.listenerRemove(id, 'touchmove', true);
        }

        upmouse=true;

        var x = nx, y = ny;

        var bordzoom = _m.$dc('zoom').getBoundingClientRect();
        var bordsvg =  _m.$dc('contsvg').getBoundingClientRect();
        
        tx = getTranslateX(_m.$dc('contsvg'));
        ty = getTranslateY(_m.$dc('contsvg'));

        if (bordsvg.left > bordzoom.left) {
            if(curScale>1){
                x = oldx-oldxs+tx;
            }
        }
        if (bordsvg.right < bordzoom.right) {
            if(curScale>1){
                x = oldx-oldxs+tx;
            }
        }
        if (bordsvg.top > bordzoom.top) {
            if(curScale>1){
                y = oldy-oldys+ty;
            }
        }
        if (bordsvg.bottom < bordzoom.bottom) {
            if(curScale>1){
                y = oldy-oldys+ty;
            }
        }
        _m.$dc('contsvg').style.mstransform = "translate(" + x + "px, " + y + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.WebkitTransform = "translate(" + x + "px, " + y + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.transform = "translate(" + x + "px, " + y + "px) scale(" + curScale + ")";
        
       alignBord(x,y);

    },

    alignBord = function(x,y){
        var bordzoom = _m.$dc('zoom').getBoundingClientRect();
        var bordsvg =  _m.$dc('contsvg').getBoundingClientRect();
         // realignement des bords
        //  console.log(bordzoom);
        //  console.log(bordsvg);
         var decTop = Math.floor((_m.$dc('svgcontainer').getBoundingClientRect().height - bordzoom.height)/2);
         var decLeft = (bordsvg.width-bordzoom.width)/2;
        //  console.log('decLeft', decLeft, 'decTop', decTop);
         //top
         if(bordsvg.top > bordzoom.top){
            //  console.log('1');
             _m.$dc('contsvg').style.transform = "translate(" + x + "px, " + decTop + "px) scale(" + curScale + ")";
         }
         //left
         if(bordsvg.left > bordzoom.left){
            //  console.log('2');
             _m.$dc('contsvg').style.transform = "translate(" + decLeft + "px, " + y + "px) scale(" + curScale + ")";
         }
         // left and top
         if(bordsvg.top > bordzoom.top && bordsvg.left > bordzoom.left){
            //  console.log('3');
             _m.$dc('contsvg').style.transform = "translate(" + decLeft + "px, " + decTop + "px) scale(" + curScale + ")";
         }
         //right
         if(bordsvg.right < bordzoom.right){
            //  console.log('4');
             _m.$dc('contsvg').style.transform = "translate(" + -decLeft + "px, " + y + "px) scale(" + curScale + ")";
         }
         // right and top
         if(bordsvg.top > bordzoom.top && bordsvg.right < bordzoom.right){
            //  console.log('5');
             _m.$dc('contsvg').style.transform = "translate(" + -decLeft + "px, " + decTop + "px) scale(" + curScale + ")";
         }
         //bottom
         if(bordsvg.bottom < bordzoom.bottom){
            //  console.log('6');
             _m.$dc('contsvg').style.transform = "translate(" + x + "px, " + -decTop + "px) scale(" + curScale + ")";
         }
         //left and bottom
         if(bordsvg.bottom < bordzoom.bottom && bordsvg.left > bordzoom.left){
            //  console.log('7');
             _m.$dc('contsvg').style.transform = "translate(" + decLeft + "px, " + -decTop + "px) scale(" + curScale + ")";
         }
          // right and bottom
          if(bordsvg.bottom < bordzoom.bottom && bordsvg.right < bordzoom.right){
            //  console.log('8');
             _m.$dc('contsvg').style.transform = "translate(" + -decLeft + "px, " + -decTop + "px) scale(" + curScale + ")";
         }
    },

    mouseDrag = function(e,c,t){
        if(upmouse) return false;

        var id = e.currentTarget.getAttribute('id');
        // console.log('id : ',id)
        var xd = 0;
        var yd = 0;
        // no touch
        if(e.changedTouches === undefined || e.changedTouches === false){
            xd = e.x === undefined ? e.pageX : e.x;
            yd = e.y === undefined ? e.pageY : e.y;
        }
        // touch
        if(e.changedTouches) {
            xd = e.changedTouches[0].clientX;
            yd = e.changedTouches[0].clientY;
        }

        // no drag
        if(xd == xs && yd == ys) return false;

        //console.log('xs2 : ',xs);

        nx = xd-xs;
        // var ratio = _m.$dc('contsvg').getBoundingClientRect().width / _m.$dc('zoom').getBoundingClientRect().width;
        ny = yd-ys;

        if(curScale>1){
            nx = xd-xs+tx;
            ny = yd-ys+ty;
        }
        
        // var zy = _m.$dc('contsvg').getBoundingClientRect().top;
        // var zx = (_m.$dc('zoom').getBoundingClientRect().left-_m.$dc('contsvg').getBoundingClientRect().left);
        _m.$dc('contsvg').style.mstransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.WebkitTransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
        _m.$dc('contsvg').style.transform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
    },
    
    // clic sur un element SVG
    clic = function(e,c,t){
        var x,y;
        if(e.changedTouches === undefined || e.changedTouches === false){
            x = e.x === undefined ? e.pageX : e.x;
            y = e.y === undefined ? e.pageY : e.y;
        }
        // touch
        if(e.changedTouches) {
            x = e.changedTouches[0].clientX;
            y = e.changedTouches[0].clientY;
        }

        if(x != oldx && y != oldy) return false;

        var id = '';
        try{
            var id = t.getAttribute('id');
        }catch (e){
            _m.listenerRemove('zoom', 'mousemove', true);
           
        }
        if(id != '' && id != null && id != 'svgcontainer') {
            // console.log('id svg', id);
            _m.$dc(id).setAttribute('style',"fill:"+curColor);
        }
        return false;
    },

    menuClic = function(e,c,t){
        // console.log(document.getElementById("mysvg").innerHTML);
        switch(e.currentTarget.getAttribute('id')){

            case 'download':
                var datasvg = document.getElementById("mysvg").innerHTML;
                // console.log(datasvg)
                // canvg(document.getElementById('exportsvg'), '<svg id="svgcontainer" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1311.974 1680.872">' + datasvg + '</svg>',{offsetX:171, offsetY:449, log:true, ignoreClear:true, renderCallback:function(){
                canvg(document.getElementById('exportsvg'), datasvg, {offsetX:171, offsetY:449, log:true, ignoreClear:true, renderCallback:function(){
                    console.log('DONE !');
                    var data = scanvas.toDataURL('image/jpeg',.8);
                    url = data.replace(/^data:image\/[^;]+/, 'data:application/octet-stream');
                    _m.$dc('imgTodownload').setAttribute('href',url);
                    _m.$dc('imgTodownload').setAttribute('download',_carac+'.jpg');
                    _m.$dc('imgTodownload').click();                            
                }});
                return false;
            break;

            case 'home':
                _m.removeAclass('intro','nodisplay');
                _m.addAclass('jeu','nodisplay');
                return false;
            break;

            case 'reset':
                // console.log('carac ',_carac);
                loadSVG(_carac);
                return false;
            break;
            
            case 'zoomin':
                if (curScale < 8) {
                    curScale += .5;
                    _m.addAclass('contsvg','forZoom');
                    // _m.$dc('contsvg').style.transform = "scale(" + curScale + ")";
                    _m.$dc('contsvg').style.msTransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    _m.$dc('contsvg').style.WebkitTransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    _m.$dc('contsvg').style.transform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    // _m.listenertransAdd('contsvg', 'transitionend', transendZoom2);
                    alignBord(nx,ny);
                }
                return false;
                break;
                
                case 'zoomout':
                if (curScale > 1){
                    curScale -= .5;
                    _m.addAclass('contsvg','forZoom');
                    _m.$dc('contsvg').style.msTransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    _m.$dc('contsvg').style.WebkitTransform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    _m.$dc('contsvg').style.transform = "translate(" + nx + "px, " + ny + "px) scale(" + curScale + ")";
                    // _m.$dc('contsvg').style.transform = "scale(" + curScale + ")";
                    // _m.listenertransAdd('contsvg', 'transitionend', transendZoom2);
                    alignBord(nx,ny);
                }
                return false;
            break;
        }

        // canvg('exportsvg','_svg/beerus.svg',{log:true,renderCallback:function(dom){
        //     console.log('DONE !');
        // }});
        return false;
    },

    colorpicker = function(e, c, t){
        curColor = e.currentTarget.getAttribute('style').slice(-7);
        // console.log('colorpicker : ', curColor);
        for(var i = 0; i < rowcolor.children.length; i++){
            if(_m.hasAclass(rowcolor.children[i].childNodes[0], 'current')) _m.removeAclass(rowcolor.children[i].childNodes[0], 'current');
        }
        _m.addAclass(e.currentTarget,'current');
    },

    deferImageCanvas = function(callback, ind, im, x, y, w, h, dx, dy, dw, dh){
        if (im.addEventListener != undefined){
            im.addEventListener('load',function(e){
                callback(im, ind, x, y, w, h, dx, dy, dw, dh);
            });
        }else if (im.readyState){ // IE8
            im.onreadystatechange = function(){
                if(im.readyState == 'loaded' || im.readyState == 'complete') {
                    callback(im, ind, x, y, w, h, dx, dy, dw, dh);
                }
            };
        }
    },

    loadInCanvas = function(){
        // ajout bg
        scontext.save();
        scontext.setTransform(1, 0, 0, 1, 0, 0);
        scontext.clearRect(0, 0, 1654, 2340);
        scontext.restore();
        var bimg = new Image();
        bimg.src = '_bg/bg_export_color.jpg';
        deferImageCanvas(loadInCanvasPlayer, 0 , bimg, 0, 0, 1654, 2340, 0, 0, 1654, 2340);
    },

    loadInCanvasPlayer = function(im, ind, x, y, w, h, dx, dy, dw, dh){
        // dessine l'image de fond 
        scontext.drawImage(im, x, y, w, h, dx, dy, dw, dh);
    },

    Init = function(_init, _c){
        // _m = manageEvents,
        // console.log('c ',_c);
        _carac = _c;
        rowcolor = _m.$dc('rowcolor');
        svgcnt = _m.$dc('svgcontainer');
        scanvas = _m.$dc('exportsvg'),
        scontext = scanvas.getContext('2d'),
        decalTop = _m.$dc('contsvg').getBoundingClientRect().top;

        if(!_init){
            _m.listenerAdd('zoom', 'touchstart', mouseDown,false);
            _m.listenerAdd('zoom', 'touchmove', function(e,c,t){return false},true);
            _m.listenerAdd('zoom', 'mousedown', mouseDown,true);
            _m.listenerAdd('zoom', 'mouseup', mouseUp, true);
            _m.listenerAdd('zoom', 'touchend', mouseUp, false);
            _m.listenertransAdd('contsvg', 'transitionend', transendZoom, true);

            _m.listenerAdd('contsvg', 'click', clic, true);

            _m.listenClass('picker', 'click', colorpicker, true);
            _m.listenClass('menugen', 'click', menuClic, true);

            _m.listenClass('btzoom','click', menuClic, true);
        }

        loadInCanvas();
    };

    window.Init = Init;
    //Init();

    // DomLoaded = function(e){
    //     Init();
    // };

    // if (window.addEventListener){
    //     window.addEventListener('DOMContentLoaded', function(){DomLoaded()});
    // }else if (window.attachEvent){ // IE8
    //     window.attachEvent('onload', function() { DomLoaded(); });
    // }else{
    //     window.onload = DomLoaded();
    // }
})(manageEvents)
````
