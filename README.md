<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Mapa Mixto — 30 Lugares para Bucear en México</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha512-sA+q3Q0gP0s8kE6+6Hc2YlFZ+gGk3qA1Z6q+Vnqz5j1s1xV9s8k1g0g==" crossorigin=""/>
  <style>
    :root{
      --navy:#032f4c; /* azul marino profesional */
      --accent:#116CB2; /* azul institucional */
      --muted:#f5f7fb;
      --card-bg: rgba(255,255,255,0.03);
    }
    html,body,#map{height:100%;margin:0;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial;}
    body{background:linear-gradient(180deg,var(--navy),#023b56);color:#fff}
    .container{display:flex;gap:16px;height:100vh;padding:20px;box-sizing:border-box}
    .panel{width:380px;max-width:40%;background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));border-radius:12px;padding:14px;box-shadow:0 6px 18px rgba(0,0,0,0.35);overflow:auto}
    .map-wrap{flex:1;border-radius:12px;overflow:hidden;box-shadow:0 6px 18px rgba(0,0,0,0.35)}
    .site-item{padding:10px;border-radius:10px;margin-bottom:10px;background:var(--card-bg);border:1px solid rgba(255,255,255,0.03);cursor:pointer}
    .site-item:hover{transform:translateY(-3px);box-shadow:0 8px 20px rgba(0,0,0,0.5)}
    h1{font-size:18px;margin:0 0 8px 0}
    .meta{font-size:13px;opacity:0.9}
    .thumb{width:100%;height:110px;object-fit:cover;border-radius:8px;margin-top:8px}
    .filters{display:flex;gap:8px;margin-bottom:12px}
    .btn{background:transparent;border:1px solid rgba(255,255,255,0.08);padding:8px 10px;border-radius:8px;color:#fff;font-size:13px;cursor:pointer}
    .search{width:100%;padding:10px;border-radius:10px;border:1px solid rgba(255,255,255,0.06);background:transparent;color:#fff;margin-bottom:10px}
    .popup-card{width:280px}
    .popup-card img{width:100%;height:140px;object-fit:cover;border-radius:6px;margin-bottom:8px}
    .credits{font-size:12px;opacity:0.7;margin-top:10px}
    @media(max-width:900px){.container{flex-direction:column}.panel{width:100%;max-height:36vh}.map-wrap{height:64vh}}
  </style>
</head>
<body>
  <div class="container">
    <div class="panel">
      <h1>30 Mejores Lugares para Bucear en México — Mapa Mixto</h1>
      <div class="meta">Haz clic en un marcador del mapa o en cualquiera de las fichas para ver detalles.</div>

      <input id="search" class="search" placeholder="Buscar sitio, región o tipo..." />
      <div class="filters">
        <button class="btn" data-filter="all">Todos</button>
        <button class="btn" data-filter="Caribe">Caribe</button>
        <button class="btn" data-filter="Pacífico">Pacífico</button>
        <button class="btn" data-filter="Cenotes">Cenotes</button>
      </div>

      <div id="list"></div>
      <div class="credits">Mapa interactivo embebible — estilo profesional azul marino. Fotos: Unsplash (imágenes por consulta dinámica).</div>
    </div>

    <div class="map-wrap" id="map"></div>
  </div>

  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
  <script>
    const SITES = [
      {id:1,name:'Cozumel',region:'Caribe',type:'Arrecife',difficulty:'Intermedio',bestMonths:'Ene-Dic',lat:20.42298,lng:-86.92234,desc:'Barrera arrecifal mesoamericana; excelente visibilidad.',img:'https://source.unsplash.com/600x400/?cozumel,reef,diving'},
      {id:2,name:'Isla Mujeres',region:'Caribe',type:'Pelágicos / MUSA',difficulty:'Principiante–Intermedio',bestMonths:'Oct-Abr',lat:21.2310,lng:-86.7316,desc:'Museo Subacuático de Arte y encuentros con tiburones gata.',img:'https://source.unsplash.com/600x400/?isla-mujeres,diving'},
      {id:3,name:'Cancún',region:'Caribe',type:'Turístico / Museo',difficulty:'Principiante',bestMonths:'Ene-Dic',lat:21.1619,lng:-86.8515,desc:'MUSA y arrecifes accesibles para todos los niveles.',img:'https://source.unsplash.com/600x400/?cancun,diving'},
      {id:4,name:'Playa del Carmen',region:'Caribe',type:'Pelágicos',difficulty:'Intermedio–Avanzado',bestMonths:'Nov-Mar',lat:20.6296,lng:-87.0739,desc:'Buena temporada de bull sharks (nov-mar).',img:'https://source.unsplash.com/600x400/?playa-del-carmen,diving'},
      {id:5,name:'Mahahual',region:'Caribe',type:'Arrecife',difficulty:'Principiante–Intermedio',bestMonths:'Nov-Jun',lat:18.7087,lng:-87.7082,desc:'Arrecifes prístinos y menos concurridos.',img:'https://source.unsplash.com/600x400/?maha-hual,reef,diving'},
      {id:6,name:'Banco Chinchorro',region:'Caribe',type:'Atolón / Naufragios',difficulty:'Intermedio',bestMonths:'Nov-May',lat:18.6072,lng:-87.6350,desc:'Atolón remoto con naufragios y rica vida marina.',img:'https://source.unsplash.com/600x400/?atoll,reef,diving'},
      {id:7,name:'Akumal',region:'Caribe',type:'Tortugas',difficulty:'Principiante',bestMonths:'Ene-Dic',lat:20.3574,lng:-87.3084,desc:'Conocido por las tortugas verdes; inmersiones suaves.',img:'https://source.unsplash.com/600x400/?akumal,turtle,diving'},
      {id:8,name:'Tulum (Cenotes)',region:'Cenotes',type:'Cenotes / Cavernas',difficulty:'Cueva (cert.)',bestMonths:'Ene-Dic',lat:20.2110,lng:-87.4650,desc:'Sistemas de cuevas, impresionantes formaciones y luz cenital.',img:'https://source.unsplash.com/600x400/?tulum,cenote,diving'},
      {id:9,name:'Cenote Dos Ojos',region:'Cenotes',type:'Cenote',difficulty:'Caverna',bestMonths:'Ene-Dic',lat:20.3830,lng:-87.3540,desc:'Uno de los sistemas de cuevas más famosos del mundo.',img:'https://source.unsplash.com/600x400/?dos-ojos,cenote'},
      {id:10,name:'Cenote Angelita',region:'Cenotes',type:'Cenote',difficulty:'Intermedio–Avanzado',bestMonths:'Ene-Dic',lat:20.3087,lng:-87.3826,desc:"Capa de sulfuro crea un efecto de 'río' subacuático.",img:'https://source.unsplash.com/600x400/?angelita,cenote'},
      {id:11,name:'Gran Cenote',region:'Cenotes',type:'Cenote',difficulty:'Principiante–Intermedio',bestMonths:'Ene-Dic',lat:20.3270,lng:-87.5450,desc:'Accesible y visualmente impresionante.',img:'https://source.unsplash.com/600x400/?gran-cenote,cenote'},
      {id:12,name:'Cenote The Pit',region:'Cenotes',type:'Cenote',difficulty:'Avanzado',bestMonths:'Ene-Dic',lat:20.2730,lng:-87.4620,desc:'Profundo, con luz cenital y efectos únicos.',img:'https://source.unsplash.com/600x400/?the-pit,cenote'},
      {id:13,name:'Puerto Morelos',region:'Caribe',type:'Arrecife',difficulty:'Principiante',bestMonths:'Ene-Dic',lat:20.8260,lng:-86.8723,desc:'Parque Nacional Arrecifes; ideal para aprender.',img:'https://source.unsplash.com/600x400/?puerto-morelos,reef'},
      {id:14,name:'Isla Contoy',region:'Caribe',type:'Arrecife / Reserva',difficulty:'Principiante–Intermedio',bestMonths:'Nov-May',lat:21.4510,lng:-86.8660,desc:'Reserva protegida con arrecifes prístinos.',img:'https://source.unsplash.com/600x400/?isla-contoy,bird,reef'},
      {id:15,name:'Cenote Calavera',region:'Cenotes',type:'Caverna',difficulty:'Intermedio',bestMonths:'Ene-Dic',lat:20.2118,lng:-87.4323,desc:'Entrada característica y experiencia vibrante.',img:'https://source.unsplash.com/600x400/?calavera,cenote'},
      {id:16,name:'Cabo Pulmo',region:'Pacífico',type:'Arrecife',difficulty:'Intermedio',bestMonths:'Nov-May',lat:23.4488,lng:-109.3996,desc:'Único arrecife vivo en la costa oeste de N. América.',img:'https://source.unsplash.com/600x400/?cabo-pulmo,reef'},
      {id:17,name:'Archipiélago de Revillagigedo (Socorro)',region:'Pacífico',type:'Pelágicos',difficulty:'Avanzado / Liveaboard',bestMonths:'Nov-May',lat:18.8394,lng:-112.9066,desc:'Pelágicos grandes: mantarrayas, tiburones martillo, tiburones.',img:'https://source.unsplash.com/600x400/?socorro,island,diving'},
      {id:18,name:'Isla Guadalupe',region:'Pacífico',type:'Jaula / Tiburones',difficulty:'Avanzado',bestMonths:'Ago-Nov',lat:29.1000,lng:-118.3000,desc:'Buceo en jaula con tiburones blancos.',img:'https://source.unsplash.com/600x400/?guadalupe,island,shark'},
      {id:19,name:'Bahía de los Ángeles',region:'Pacífico',type:'Pelágicos',difficulty:'Intermedio',bestMonths:'Jul-Sep',lat:28.9540,lng:-113.5600,desc:'Avistamiento de tiburón ballena y fauna rica.',img:'https://source.unsplash.com/600x400/?bahia-los-angeles,diving'},
      {id:20,name:'La Paz',region:'Pacífico',type:'Fauna / Lobos Marinos',difficulty:'Principiante–Intermedio',bestMonths:'Oct-May',lat:24.1426,lng:-110.3106,desc:'Lobos marinos, mantarrayas y tiburones ballena en temporada.',img:'https://source.unsplash.com/600x400/?la-paz,diving'},
      {id:21,name:'Cabo San Lucas',region:'Pacífico',type:'Arrecife / Pelágicos',difficulty:'Intermedio',bestMonths:'Nov-Mar',lat:22.8894,lng:-109.9167,desc:'Formaciones rocosas y buceo pelágico.',img:'https://source.unsplash.com/600x400/?cabo-san-lucas,diving'},
      {id:22,name:'Loreto',region:'Pacífico',type:'Parque Marino',difficulty:'Intermedio',bestMonths:'Oct-May',lat:26.0112,lng:-111.3439,desc:'Parque Nacional Marino con rica biodiversidad.',img:'https://source.unsplash.com/600x400/?loreto,mexico,diving'},
      {id:23,name:'Puerto Vallarta',region:'Pacífico',type:'Arrecife',difficulty:'Principiante–Intermedio',bestMonths:'Nov-Jun',lat:20.6534,lng:-105.2253,desc:'Buceo accesible y vida marina diversa.',img:'https://source.unsplash.com/600x400/?puerto-vallarta,diving'},
      {id:24,name:'Islas Marietas',region:'Pacífico',type:'Rocas / Cuevas',difficulty:'Principiante–Intermedio',bestMonths:'Nov-May',lat:20.7098,lng:-105.5049,desc:'Formaciones rocosas únicas y biodiversidad marina.',img:'https://source.unsplash.com/600x400/?marietas,islands,diving'},
      {id:25,name:'Manzanillo',region:'Pacífico',type:'Naufragios',difficulty:'Intermedio',bestMonths:'Nov-May',lat:19.0550,lng:-104.3167,desc:'Naufragios y vida pelágica.',img:'https://source.unsplash.com/600x400/?manzanillo,diving'},
      {id:26,name:'Huatulco',region:'Pacífico',type:'Bahías Protegidas',difficulty:'Principiante–Intermedio',bestMonths:'Nov-May',lat:15.8470,lng:-96.1350,desc:'Bahías con corales blandos y peces tropicales.',img:'https://source.unsplash.com/600x400/?huatulco,diving'},
      {id:27,name:'Zihuatanejo / Ixtapa',region:'Pacífico',type:'Arrecifes',difficulty:'Principiante–Intermedio',bestMonths:'Nov-May',lat:17.6400,lng:-101.5480,desc:'Arrecifes tranquilos y tortugas.',img:'https://source.unsplash.com/600x400/?zihuatanejo,diving'},
      {id:28,name:'Isla Isabel',region:'Pacífico',type:'Arrecife',difficulty:'Intermedio',bestMonths:'Nov-May',lat:21.8810,lng:-105.2210,desc:'Isla volcánica con arrecifes y aves marinas.' ,img:'https://source.unsplash.com/600x400/?isla-isabel,diving'},
      {id:29,name:'Mazatlán',region:'Pacífico',type:'Pacífico / Naufragios',difficulty:'Intermedio',bestMonths:'Nov-May',lat:23.2494,lng:-106.4111,desc:'Buceo en el Pacífico con naufragios y fauna grande.',img:'https://source.unsplash.com/600x400/?mazatlan,diving'},
      {id:30,name:'Bahía Magdalena',region:'Pacífico',type:'Ballenas / Observación',difficulty:'Principiante',bestMonths:'Feb-Apr',lat:24.5000,lng:-112.0000,desc:'Zona reconocida por ballenas grises y snorkel con fauna.' ,img:'https://source.unsplash.com/600x400/?bahia-magdalena,whales'}
    ];

    // Initialize map
    const map = L.map('map', {zoomControl:true}).setView([23.6345,-102.5528],5);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{maxZoom:18,attribution:'© OpenStreetMap contributors'}).addTo(map);

    const markers = {};

    function makePopup(site){
      return `
        <div class="popup-card">
          <img src="${site.img}" alt="${site.name}" onerror="this.style.display='none'" />
          <strong>${site.name}</strong>
          <div style="font-size:13px;margin-top:6px">${site.region} • ${site.type}</div>
          <div style="font-size:13px;margin-top:6px">Dificultad: ${site.difficulty} • Mejores meses: ${site.bestMonths}</div>
          <p style="font-size:13px;margin-top:8px;color:#eef;">${site.desc}</p>
        </div>
      `;
    }

    // Add markers & list
    const listEl = document.getElementById('list');
    SITES.forEach(site=>{
      const m = L.marker([site.lat,site.lng]).addTo(map).bindPopup(makePopup(site));
      markers[site.id]=m;

      const item = document.createElement('div');
      item.className='site-item';
      item.innerHTML = `<strong>${site.name}</strong><div class="meta">${site.region} • ${site.type}</div><img class="thumb" src="${site.img}" alt="${site.name}" onerror="this.style.display='none'" />`;
      item.onclick = ()=>{map.setView([site.lat,site.lng],10); m.openPopup();};
      item.dataset.region = site.region;
      item.dataset.name = site.name.toLowerCase();
      item.dataset.type = site.type.toLowerCase();
      listEl.appendChild(item);
    });

    // Filtering
    document.querySelectorAll('.btn').forEach(b=>{
      b.addEventListener('click',()=>{
        const f = b.dataset.filter;
        const items = Array.from(document.querySelectorAll('.site-item'));
        items.forEach(i=>{
          if(f==='all' || i.dataset.region===f || i.dataset.region.toLowerCase()===f.toLowerCase()) i.style.display='block'; else i.style.display='none';
        });
      });
    });

    // Search
    document.getElementById('search').addEventListener('input',e=>{
      const q = e.target.value.toLowerCase();
      document.querySelectorAll('.site-item').forEach(i=>{
        const name = i.dataset.name; const type=i.dataset.type;
        if(name.includes(q) || type.includes(q)) i.style.display='block'; else i.style.display='none';
      });
    });

    // Fit bounds to markers
    const group = L.featureGroup(Object.values(markers));
    map.fitBounds(group.getBounds().pad(0.2));

    // Make the map friendly when embedded (disable scroll zoom by default)
    map.scrollWheelZoom.disable();
  </script>
</body>
</html>
