<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>OYO — Hotel Performance Dashboard</title>

  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <!-- Leaflet for maps -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <style>
    :root{
      --brand-red: #ff3b30;
      --brand-dark: #2b2b2b;
      --muted: #6b6b6b;
      --bg: #f7f7f8;
      --card: #ffffff;
      --glass: rgba(255,255,255,0.85);
      --radius: 14px;
      font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
    }
    *{box-sizing:border-box}
    body{
      margin:0;
      background:linear-gradient(180deg,#fbfbfc 0%, #f2f2f3 100%);
      color:var(--brand-dark);
      -webkit-font-smoothing:antialiased;
      -moz-osx-font-smoothing:grayscale;
    }

    /* Layout */
    .app {
      display: grid;
      grid-template-columns: 260px 1fr;
      gap: 24px;
      min-height:100vh;
    }

    /* Sidebar */
    .sidebar{
      background: linear-gradient(180deg,var(--brand-red), #d7261c);
      color:white;
      padding:28px;
      border-radius: 18px;
      margin:20px;
      display:flex;
      flex-direction:column;
      gap:18px;
      box-shadow: 0 8px 30px rgba(0,0,0,0.08);
    }
    .logo {
      display:flex; align-items:center; gap:12px;
    }
    .logo .mark {
      width:56px; height:56px; background:white; color:var(--brand-red);
      font-weight:700; display:flex; align-items:center; justify-content:center;
      border-radius:12px; font-size:20px;
      box-shadow: 0 6px 20px rgba(0,0,0,0.12);
    }
    .logo h2{font-size:18px;margin:0; letter-spacing:0.2px}
    .nav{margin-top:8px; display:flex; flex-direction:column; gap:8px}
    .nav button{
      background:transparent; border:0; color:rgba(255,255,255,0.95);
      text-align:left; padding:12px; border-radius:10px; cursor:pointer; font-weight:600;
    }
    .nav button:hover{ background: rgba(255,255,255,0.06) }

    .filter-group{
      background: rgba(255,255,255,0.06); padding:12px; border-radius:10px;
    }
    .filter-group label{ display:block; font-size:13px; color: rgba(255,255,255,0.9); margin-bottom:6px; }
    select,input[type="range"]{ width:100%; padding:8px 10px; border-radius:8px; border:none; outline:none; }

    /* Main content */
    .main{
      padding:28px 20px;
      display:flex;
      flex-direction:column;
      gap:18px;
      margin:20px 20px 40px 0;
    }
    .header{
      display:flex; justify-content:space-between; align-items:center;
    }
    .title{ font-size:26px; font-weight:700; }
    .date-filter{
      background:var(--card); padding:8px 12px; border-radius:12px; box-shadow: 0 6px 18px rgba(0,0,0,0.04);
    }

    /* KPI cards */
    .kpis{
      display:grid;
      grid-template-columns: repeat(4,1fr);
      gap:14px;
    }
    .kpi{
      background:var(--card); padding:18px; border-radius:12px; box-shadow: 0 6px 20px rgba(16,24,40,0.06);
      transition: transform .28s ease;
    }
    .kpi:hover{ transform: translateY(-6px) }
    .kpi .label{ color:var(--muted); font-size:13px; margin-bottom:8px }
    .kpi .value{ font-size:22px; font-weight:800; }

    /* Cards for charts */
    .grid-3{
      display:grid;
      grid-template-columns: 1.2fr 0.8fr 0.9fr;
      gap:14px;
    }
    .card{ background:var(--card); padding:16px; border-radius:12px; box-shadow: 0 6px 20px rgba(16,24,40,0.04); }
    .card h3{ margin:0 0 10px 0; font-weight:700; font-size:16px }

    /* Table */
    .table-card{ padding:10px 14px; }
    table{ width:100%; border-collapse:collapse; }
    thead th{ text-align:left; padding:12px 8px; color:var(--muted); font-weight:600; font-size:13px }
    tbody tr{ background:transparent; border-top:1px solid #f1f2f3 }
    tbody td{ padding:12px 8px; font-size:14px; vertical-align:middle }

    .sparkline{ width:110px; height:36px; }

    /* Map container */
    #map{ width:100%; height:360px; border-radius:10px; overflow:hidden }

    /* Responsive */
    @media (max-width: 1000px){
      .app{ grid-template-columns: 1fr; }
      .sidebar{ flex-direction:row; gap:12px; align-items:center; padding:12px; margin:12px; border-radius:12px }
      .main{ margin:12px 12px 40px 12px; padding:14px }
      .kpis{ grid-template-columns: repeat(2,1fr) }
      .grid-3{ grid-template-columns: 1fr; }
    }

    /* subtle entrance animation */
    .fade-in { opacity: 0; transform: translateY(8px); animation: fadeInUp .6s forwards; }
    @keyframes fadeInUp { to {opacity:1; transform:none} }
  </style>
</head>
<body>
  <div class="app">
    <!-- SIDEBAR -->
    <aside class="sidebar fade-in" style="animation-delay:.05s">
      <div class="logo">
        <div class="mark">OYO</div>
        <div>
          <h2>OYO Insights</h2>
          <div style="font-size:12px; opacity:0.9">Hotel Performance</div>
        </div>
      </div>

      <nav class="nav">
        <button>Filters</button>
        <button>Location</button>
        <button>Price Range</button>
        <button>Rating Range</button>
      </nav>

      <div style="flex:1"></div>

      <div class="filter-group">
        <label>Location</label>
        <select id="filter-location" multiple size="4" style="height:110px;">
          <!-- options will be populated -->
        </select>
      </div>

      <div class="filter-group">
        <label>Price (max) ₹<span id="price-label">4000</span></label>
        <input id="filter-price" type="range" min="500" max="6000" step="100" value="4000">
      </div>

      <div class="filter-group">
        <label>Min Rating <span id="rating-label">4.0</span></label>
        <input id="filter-rating" type="range" min="1" max="5" step="0.1" value="4.0">
      </div>

      <div style="font-size:12px; opacity:0.9; margin-top:10px">Data source: <strong>OYO_HOTEL_ROOMS.csv</strong></div>
    </aside>

    <!-- MAIN -->
    <main class="main">
      <div class="header fade-in" style="animation-delay:.1s">
        <div class="title">Hotel Performance Dashboard</div>
        <div class="date-filter">
          <label style="font-size:13px; color:var(--muted);">Date range</label>
          <select id="date-range" style="border:0; background:transparent; font-weight:600">
            <option>Last 30 days</option>
            <option>Last 90 days</option>
            <option>This year</option>
          </select>
        </div>
      </div>

      <!-- KPIs -->
      <section class="kpis fade-in" style="animation-delay:.14s">
        <div class="kpi">
          <div class="label">Average Price per Room</div>
          <div class="value" id="kpi-price">₹0</div>
        </div>
        <div class="kpi">
          <div class="label">Average Discount</div>
          <div class="value" id="kpi-discount">0%</div>
        </div>
        <div class="kpi">
          <div class="label">Average Rating</div>
          <div class="value" id="kpi-rating">0.0</div>
        </div>
        <div class="kpi">
          <div class="label">Total Hotels Listed</div>
          <div class="value" id="kpi-count">0</div>
        </div>
      </section>

      <!-- Charts + Map -->
      <section class="grid-3">
        <div class="card fade-in" style="animation-delay:.18s">
          <h3>Average Price by Location</h3>
          <canvas id="barChart" height="160"></canvas>
        </div>

        <div class="card fade-in" style="animation-delay:.22s">
          <h3>Distribution by Discount Range</h3>
          <canvas id="pieChart" height="160"></canvas>
        </div>

        <div class="card fade-in" style="animation-delay:.26s">
          <h3>Trend: Ratings vs Price</h3>
          <canvas id="lineChart" height="160"></canvas>
        </div>
      </section>

      <section class="card fade-in" style="animation-delay:.28s">
        <h3>Hotel Locations</h3>
        <div id="map"></div>
      </section>

      <section class="card table-card fade-in" style="animation-delay:.32s">
        <h3>Hotel Performance</h3>
        <div style="overflow:auto">
          <table id="hotel-table">
            <thead>
              <tr>
                <th>Hotel Name</th>
                <th>Location</th>
                <th>Price</th>
                <th>Discount</th>
                <th>Rating</th>
                <th>Trend</th>
              </tr>
            </thead>
            <tbody>
              <!-- rows inserted dynamically -->
            </tbody>
          </table>
        </div>
      </section>
    </main>
  </div>

  <script>
    /***********************
     * Sample dataset (12 rows)
     * Replace this array with your CSV / JSON parse.
     ***********************/
    const hotels = [
      { id:1, name:"OYO Townhouse", city:"Gurgaon", lat:28.4595, lon:77.0266, price:1600, discount:40, rating:4.8, trend:[4.6,4.7,4.8,4.7,4.8] },
      { id:2, name:"OYO Flagship", city:"Hyderabad", lat:17.3850, lon:78.4867, price:2300, discount:50, rating:4.7, trend:[4.5,4.6,4.7,4.7,4.7] },
      { id:3, name:"OYO Rooms", city:"Mumbai", lat:19.0760, lon:72.8777, price:2400, discount:60, rating:4.5, trend:[4.4,4.5,4.5,4.6,4.5] },
      { id:4, name:"OYO Townhouse", city:"Bangalore", lat:12.9716, lon:77.5946, price:1300, discount:40, rating:5.0, trend:[4.9,5.0,5.0,4.9,5.0] },
      { id:5, name:"OYO Select", city:"Chennai", lat:13.0827, lon:80.2707, price:2100, discount:55, rating:4.6, trend:[4.4,4.5,4.6,4.6,4.6] },
      { id:6, name:"OYO Cozy", city:"Hyderabad", lat:17.3850, lon:78.4867, price:1800, discount:30, rating:4.1, trend:[4.0,4.1,4.1,4.1,4.1] },
      { id:7, name:"OYO Prime", city:"Mumbai", lat:19.0760, lon:72.8777, price:2800, discount:70, rating:4.3, trend:[4.2,4.3,4.3,4.4,4.3] },
      { id:8, name:"OYO Inn", city:"Bangalore", lat:12.9716, lon:77.5946, price:2700, discount:65, rating:4.2, trend:[4.1,4.2,4.2,4.3,4.2] },
      { id:9, name:"OYO Stay", city:"Gurgaon", lat:28.4595, lon:77.0266, price:1900, discount:45, rating:4.4, trend:[4.3,4.4,4.4,4.4,4.4] },
      { id:10, name:"OYO Central", city:"Chennai", lat:13.0827, lon:80.2707, price:2600, discount:60, rating:4.9, trend:[4.7,4.8,4.9,4.9,4.9] },
      { id:11, name:"OYO Budget", city:"Hyderabad", lat:17.3850, lon:78.4867, price:1200, discount:20, rating:3.8, trend:[3.7,3.8,3.8,3.9,3.8] },
      { id:12, name:"OYO Luxe", city:"Mumbai", lat:19.0760, lon:72.8777, price:3500, discount:80, rating:4.6, trend:[4.5,4.6,4.6,4.7,4.6] }
    ];

    /***********************
     * Utility helpers
     ***********************/
    function distinct(arr, key){
      return Array.from(new Set(arr.map(x=>x[key]))).sort();
    }

    function formatINR(n){
      return "₹" + n.toLocaleString('en-IN');
    }

    function avg(values){
      if(!values.length) return 0;
      return values.reduce((a,b)=>a+b,0)/values.length;
    }

    function clamp(n, min, max){ return Math.max(min, Math.min(max, n)); }

    /***********************
     * Populate filters
     ***********************/
    const locSelect = document.getElementById('filter-location');
    const locations = distinct(hotels, 'city');
    locations.forEach(city=>{
      const opt = document.createElement('option');
      opt.value = city; opt.textContent = city;
      locSelect.appendChild(opt);
    });

    // Allow multi-select with ctrl/shift — but let's default to all selected
    for(let i=0;i<locSelect.options.length;i++) locSelect.options[i].selected = true;

    // Price & rating controls
    const priceRange = document.getElementById('filter-price');
    const priceLabel = document.getElementById('price-label');
    priceRange.value = 4000;
    priceLabel.textContent = priceRange.value;

    const ratingRange = document.getElementById('filter-rating');
    const ratingLabel = document.getElementById('rating-label');

    ratingLabel.textContent = ratingRange.value;

    // KPIs DOM
    const kpiPrice = document.getElementById('kpi-price');
    const kpiDiscount = document.getElementById('kpi-discount');
    const kpiRating = document.getElementById('kpi-rating');
    const kpiCount = document.getElementById('kpi-count');

    /***********************
     * Charts initialization
     ***********************/
    // Bar Chart: Average price by location
    const barCtx = document.getElementById('barChart').getContext('2d');
    const barChart = new Chart(barCtx, {
      type:'bar',
      data: { labels: [], datasets: [{ label:'Avg Price', data: [], backgroundColor: 'rgba(255,59,48,0.9)', borderRadius:8 }] },
      options: {
        responsive:true,
        plugins:{ legend:{display:false} },
        scales:{ y:{ beginAtZero:true } },
        animation:{ duration:800 }
      }
    });

    // Pie Chart: distribution by discount range
    const pieCtx = document.getElementById('pieChart').getContext('2d');
    const pieChart = new Chart(pieCtx, {
      type:'doughnut',
      data:{ labels: ['Low (<40%)','Medium (40-60%)','High (>60%)'], datasets:[{ data:[], backgroundColor:['#ffd1cf','#ff6b61','#ff2f1e'] }] },
      options:{ responsive:true, plugins:{ legend:{position:'bottom'} } }
    });

    // Line Chart: Ratings vs Price (scatter-like line)
    const lineCtx = document.getElementById('lineChart').getContext('2d');
    const lineChart = new Chart(lineCtx, {
      type:'line',
      data:{ labels: [], datasets:[{ label:'Rating', data: [], tension:0.35, pointRadius:4, borderWidth:2, borderColor:'rgba(255,59,48,0.95)', fill:false }] },
      options:{ responsive:true, scales:{ y:{ min:1, max:5 } }, plugins:{ legend:{display:false} } }
    });

    /***********************
     * Leaflet map
     ***********************/
    const map = L.map('map', { zoomControl:true, attributionControl:false }).setView([20.5,78], 5);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 }).addTo(map);
    let markersLayer = L.layerGroup().addTo(map);

    /***********************
     * Table
     ***********************/
    const tableBody = document.querySelector('#hotel-table tbody');

    /***********************
     * Render / Update pipeline
     ***********************/
    function getFilters(){
      const selected = Array.from(locSelect.selectedOptions).map(o=>o.value);
      const priceMax = Number(priceRange.value);
      const minRating = Number(ratingRange.value);
      return { cities: selected, priceMax, minRating };
    }

    function filterData(){
      const f = getFilters();
      return hotels.filter(h => f.cities.includes(h.city) && h.price <= f.priceMax && h.rating >= f.minRating);
    }

    function updateKPIs(filtered){
      const avgPrice = Math.round(avg(filtered.map(d=>d.price)));
      const avgDiscount = Math.round(avg(filtered.map(d=>d.discount)));
      const avgRatingVal = +(avg(filtered.map(d=>d.rating))).toFixed(2);
      const count = filtered.length;

      // Animated counters
      animateNumber(kpiPrice, avgPrice, (v)=>formatINR(v), 700);
      animateNumber(kpiDiscount, avgDiscount, (v)=>v + "%", 700);
      animateNumber(kpiRating, avgRatingVal, (v)=> v.toFixed(2), 700);
      animateNumber(kpiCount, count, (v)=> v , 700);
    }

    function animateNumber(el, target, formatter = v => v, ms = 700){
      const start = +(el.dataset.currentValue || 0);
      const end = +target;
      el.dataset.currentValue = end;
      const startTime = performance.now();
      function tick(now){
        const t = Math.min(1, (now - startTime)/ms);
        const value = start + (end - start) * easeOutCubic(t);
        el.textContent = formatter( Math.round(value*100)/100 );
        if(t < 1) requestAnimationFrame(tick);
      }
      requestAnimationFrame(tick);
    }
    function easeOutCubic(x){ return 1 - Math.pow(1 - x, 3); }

    function updateCharts(filtered){
      // Bar chart: avg price by location
      const byCity = {};
      filtered.forEach(h => {
        byCity[h.city] = byCity[h.city] || {sum:0, count:0};
        byCity[h.city].sum += h.price;
        byCity[h.city].count += 1;
      });
      const labels = Object.keys(byCity);
      const data = labels.map(l => Math.round(byCity[l].sum / byCity[l].count));
      barChart.data.labels = labels;
      barChart.data.datasets[0].data = data;
      barChart.update();

      // Pie: discount buckets
      const buckets = { low:0, medium:0, high:0 };
      filtered.forEach(h=>{
        if(h.discount < 40) buckets.low++;
        else if(h.discount <= 60) buckets.medium++;
        else buckets.high++;
      });
      pieChart.data.datasets[0].data = [buckets.low, buckets.medium, buckets.high];
      pieChart.update();

      // Line: ratings vs price (we'll sort by price)
      const sorted = filtered.slice().sort((a,b)=>a.price-b.price);
      lineChart.data.labels = sorted.map(s => formatINR(s.price));
      lineChart.data.datasets[0].data = sorted.map(s => s.rating);
      lineChart.update();
    }

    function updateMap(filtered){
      markersLayer.clearLayers();
      filtered.forEach(h=>{
        const m = L.circleMarker([h.lat, h.lon], {
          radius: 8, color: 'rgba(255,59,48,0.9)', fillColor:'rgba(255,59,48,0.9)', fillOpacity:0.85
        }).bindPopup(`<strong>${h.name}</strong><br>${h.city}<br>${formatINR(h.price)} • ${h.discount}% off • ${h.rating}⭐`);
        markersLayer.addLayer(m);
      });
      if(filtered.length) {
        const group = L.featureGroup(markersLayer.getLayers());
        map.fitBounds(group.getBounds().pad(0.6));
      } else {
        map.setView([20.5,78], 5);
      }
    }

    function updateTable(filtered){
      tableBody.innerHTML = '';
      filtered.forEach(h=>{
        const tr = document.createElement('tr');
        tr.innerHTML = `
          <td style="font-weight:700">${h.name}</td>
          <td>${h.city}</td>
          <td>${formatINR(h.price)}</td>
          <td>${h.discount}%</td>
          <td>${h.rating.toFixed(1)}</td>
          <td><canvas class="sparkline" data-id="${h.id}"></canvas></td>
        `;
        tableBody.appendChild(tr);
      });

      // draw sparklines
      document.querySelectorAll('.sparkline').forEach(c => {
        const id = Number(c.dataset.id);
        const hotel = hotels.find(h => h.id === id);
        if(!hotel) return;
        // create small chart
        new Chart(c.getContext('2d'), {
          type:'line',
          data:{ labels: hotel.trend.map((_,i)=>i), datasets:[{ data: hotel.trend, tension:0.4, borderWidth:1.5, pointRadius:0, fill:false }] },
          options:{ responsive:false, maintainAspectRatio:false, plugins:{ legend:{display:false} }, scales:{x:{display:false}, y:{display:false}}}
        });
      });
    }

    function pipeline(){
      const filtered = filterData();
      updateKPIs(filtered);
      updateCharts(filtered);
      updateMap(filtered);
      updateTable(filtered);
    }

    /***********************
     * Events
     ***********************/
    locSelect.addEventListener('change', pipeline);
    priceRange.addEventListener('input', ()=>{
      priceLabel.textContent = priceRange.value;
      pipeline();
    });
    ratingRange.addEventListener('input', ()=>{
      ratingLabel.textContent = Number(ratingRange.value).toFixed(1);
      pipeline();
    });

    // init
    pipeline();

   
  </script>
</body>
</html>
