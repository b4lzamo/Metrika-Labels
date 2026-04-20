# Установка букмарклета для подписей чисел на графике Яндекс Метрики

Этот букмарклет добавляет числовые значения к линиям на графике в Метрике и работает по нажатию как переключатель:
- первое нажатие: `ON` (подписи включены),
- второе нажатие: `OFF` (подписи выключены).

Поддерживаемые домены:
- `metrika.yandex.ru`
- `metrika.yandex.com`

## Что делает скрипт

- Показывает числовые значения по каждой точке (дню) на линиях графика в Яндекс Метрике.
- Работает для всех видимых линий/источников на графике.
- Автоматически обновляет подписи при изменении графика (например, если включить новый источник в легенде).
- Разводит подписи по вертикали, чтобы уменьшить наложение друг на друга.
- Повторное нажатие на закладку отключает подписи и очищает нарисованные элементы.
- Индикатор в левом нижнем углу (`Числа: ON/OFF`) автоматически скрывается через 2 секунды после включения или выключения.

## 1) Скопируйте код букмарклета (актуальная версия V7)

Скопируйте строку целиком (начиная с `javascript:`):

```javascript
javascript:(()=>{if(!/metrika\.yandex\.(ru|com)$/i.test(location.hostname)){alert("Открой metrika.yandex.ru или metrika.yandex.com");return;}const KEY="__ymSvgLabelsLiveV7";const ATTR="data-ym-svg-labels";const BADGE_ID="ym-svg-labels-badge";const S=window[KEY]||(window[KEY]={on:false,mo:null,timer:null,raf:0,badgeTimer:null});const nf=new Intl.NumberFormat("ru-RU");const parseNum=t=>{if(!t)return null;const raw=String(t).toLowerCase().replace(/\u00a0/g," ");let m=1;if(/тыс|k\b|к\b/.test(raw))m=1e3;else if(/млн|mn|mm|m\b/.test(raw))m=1e6;const mm=raw.replace(/\s+/g,"").match(/-?\d+(?:[.,]\d+)?/);if(!mm)return null;const v=Number(mm[0].replace(",","."));return Number.isFinite(v)?v*m:null;};const parseTranslate=t=>{const m=String(t||"").match(/translate\(([-\d.]+)[ ,]([-\d.]+)\)/);return m?{x:+m[1],y:+m[2]}:{x:0,y:0};};const parseD=d=>{const out=[];const re=/[ML]\s*([-\d.]+)\s+([-\d.]+)/g;let m;while((m=re.exec(d||"")))out.push({x:+m[1],y:+m[2]});return out;};const remove=()=>document.querySelectorAll("g["+ATTR+"='1']").forEach(n=>n.remove());const badge=t=>{let b=document.getElementById(BADGE_ID);if(!b){b=document.createElement("div");b.id=BADGE_ID;b.style.cssText="position:fixed;right:12px;bottom:12px;z-index:2147483647;padding:6px 10px;background:#111;color:#fff;border-radius:8px;font:12px/1.2 -apple-system,BlinkMacSystemFont,Segoe UI,Arial,sans-serif;opacity:.9;";document.body.appendChild(b);}b.textContent=t;if(S.badgeTimer)clearTimeout(S.badgeTimer);S.badgeTimer=setTimeout(()=>{const bx=document.getElementById(BADGE_ID);if(bx)bx.remove();S.badgeTimer=null;},2000);};const fit=p=>{const n=p.length;if(n<2)return null;let sx=0,sy=0,sxx=0,sxy=0;for(const a of p){sx+=a.y;sy+=a.v;sxx+=a.y*a.y;sxy+=a.y*a.v;}const den=n*sxx-sx*sx;if(!den)return null;const k=(n*sxy-sx*sy)/den;return{k,b:(sy-k*sx)/n};};const renderOne=container=>{const svg=container.querySelector("svg.highcharts-root");if(!svg)return 0;const plot=svg.querySelector("rect.highcharts-plot-background");if(!plot)return 0;const plotTop=+plot.getAttribute("y"),plotH=+plot.getAttribute("height"),plotBottom=plotTop+plotH;const yGrid=[...svg.querySelectorAll(".highcharts-yaxis-grid .highcharts-grid-line")].map(p=>{const m=(p.getAttribute("d")||"").match(/M\s*[-\d.]+\s+([-\d.]+)/);return m?+m[1]:null;}).filter(Number.isFinite).sort((a,b)=>a-b);if(yGrid.length<2)return 0;const yDiv=container.querySelector("div.highcharts-axis-labels.highcharts-yaxis-labels");if(!yDiv)return 0;const yLabels=[...yDiv.querySelectorAll("span")].map(s=>({top:parseFloat(s.style.top||"NaN"),v:parseNum(s.textContent)})).filter(x=>Number.isFinite(x.top)&&x.v!==null).sort((a,b)=>a.top-b.top);if(yLabels.length<2)return 0;const n=Math.min(yGrid.length,yLabels.length);const pairs=[];for(let i=0;i<n;i++)pairs.push({y:yGrid[i],v:yLabels[i].v});const map=fit(pairs);if(!map)return 0;const series=[...svg.querySelectorAll("g.highcharts-series.highcharts-line-series")].filter(g=>g.getAttribute("visibility")!=="hidden"&&g.getAttribute("opacity")!=="0"&&g.querySelector("path.highcharts-graph"));if(!series.length)return 0;const buckets=new Map();series.forEach(g=>{const tr=parseTranslate(g.getAttribute("transform"));const path=g.querySelector("path.highcharts-graph");const color=path.getAttribute("stroke")||"#333";const pts=parseD(path.getAttribute("d")||"");pts.forEach((p,idx)=>{const x=p.x+tr.x,y=p.y+tr.y;let v=map.k*y+map.b;if(!Number.isFinite(v))return;if(v<0)v=0;const a=buckets.get(idx)||[];a.push({x,val:Math.round(v),color,ty:y-10});buckets.set(idx,a);});});const layer=document.createElementNS("http://www.w3.org/2000/svg","g");layer.setAttribute(ATTR,"1");let count=0;const minGap=13;for(const arr of buckets.values()){arr.sort((a,b)=>a.ty-b.ty);for(let i=1;i<arr.length;i++)if(arr[i].ty-arr[i-1].ty<minGap)arr[i].ty=arr[i-1].ty+minGap;const minY=plotTop+12,maxY=plotBottom-2;if(arr.length){if(arr[arr.length-1].ty>maxY){const d=arr[arr.length-1].ty-maxY;for(let i=0;i<arr.length;i++)arr[i].ty-=d;}if(arr[0].ty<minY){const d=minY-arr[0].ty;for(let i=0;i<arr.length;i++)arr[i].ty+=d;}}arr.forEach(p=>{const t=document.createElementNS("http://www.w3.org/2000/svg","text");t.setAttribute("x",String(p.x));t.setAttribute("y",String(p.ty));t.setAttribute("text-anchor","middle");t.setAttribute("font-size","12");t.setAttribute("font-weight","700");t.setAttribute("fill",p.color);t.setAttribute("stroke","#fff");t.setAttribute("stroke-width","2.6");t.setAttribute("paint-order","stroke");t.textContent=nf.format(p.val);layer.appendChild(t);count++;});}if(count)svg.appendChild(layer);return count;};const render=()=>{remove();if(!S.on)return 0;let total=0;document.querySelectorAll(".highcharts-container").forEach(c=>{try{total+=renderOne(c);}catch(e){}});return total;};const schedule=()=>{if(S.raf)return;S.raf=requestAnimationFrame(()=>{S.raf=0;render();});};S.on=!S.on;if(S.on){const total=render();badge("Числа: ON ("+total+")");if(S.mo)S.mo.disconnect();S.mo=new MutationObserver(schedule);S.mo.observe(document.body,{childList:true,subtree:true,attributes:true});if(S.timer)clearInterval(S.timer);S.timer=setInterval(render,900);}else{if(S.mo){S.mo.disconnect();S.mo=null;}if(S.timer){clearInterval(S.timer);S.timer=null;}remove();badge("Числа: OFF");}})();
```

## 2) Создайте закладку

### Вариант A (рекомендуется): через `installer.html`
1. Откройте файл [installer.html] в браузере.
2. Включите панель закладок (`Ctrl+Shift+B` на Windows/Linux, `Cmd+Shift+B` на macOS).
3. Во 2-м пункте на странице перетащите кнопку `Metrika Labels` на панель закладок.
4. Откройте Метрику и нажмите закладку для включения/выключения подписей.

### Вариант B: вручную вставить код букмарклета

### Chrome / Edge
1. Нажмите `Ctrl + Shift + O` (менеджер закладок).
2. Нажмите `Добавить новую закладку`.
3. Имя: например `Metrika Labels`.
4. В поле URL вставьте скопированный код букмарклета.
5. Сохраните.

### Firefox
1. Создайте любую новую закладку.
2. Откройте ее свойства (`Изменить`).
3. В поле `Адрес` вставьте код букмарклета.
4. Сохраните.

### Linux (Chrome / Chromium / Firefox / Edge)
1. Включите панель закладок (`Ctrl + Shift + B`).
2. Создайте новую закладку на панели.
3. Вставьте код букмарклета в поле URL/Адрес (должен начинаться с `javascript:`).
4. Сохраните.

### macOS (Chrome / Edge)
1. Откройте менеджер закладок: `⌥ Option + ⌘ Command + B`.
2. Создайте новую закладку.
3. Вставьте код букмарклета в поле URL.
4. Сохраните.

### macOS (Firefox)
1. Включите панель закладок: `⌘ Command + Shift + B`.
2. Создайте новую закладку.
3. Откройте свойства закладки и вставьте код в поле `Location/Адрес`.
4. Сохраните.

### macOS (Safari)
1. Включите строку избранного: `View` → `Show Favorites Bar`.
2. Создайте обычную закладку (любой сайт), затем откройте `Edit Bookmarks`.
3. Найдите созданную закладку и замените ее адрес на код букмарклета (`javascript:...`).
4. Сохраните изменения.

## 3) Использование

1. Откройте страницу Метрики на `metrika.yandex.ru` или `metrika.yandex.com`.
2. Убедитесь, что выбран линейный график (`По дням`).
3. Нажмите закладку:
- первый клик: включение подписей,
- второй клик: выключение.

Если добавить/включить новую поисковую систему в легенде, подписи появятся автоматически без перезагрузки страницы.

## 4) Если не работает

- Проверьте, что код в URL закладки начинается с `javascript:`.
- Проверьте домен страницы (`metrika.yandex.ru` или `metrika.yandex.com`).
- Убедитесь, что это именно линейный график (Highcharts).
- Попробуйте обновить страницу Метрики и снова нажать закладку.
