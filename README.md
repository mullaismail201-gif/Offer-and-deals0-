# Offer-and-deals0-
<!DOCTYPE html>
<html lang="hi">
<head>
  <meta charset="UTF-8">
  <title>Deals Widget</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 10px;
      background: #f9f9f9;
      color: #333;
      transition: 0.3s;
    }
    body.dark { background: #1e1e1e; color: #eee; }

    .controls {
      display: flex;
      gap: 10px;
      margin-bottom: 10px;
      flex-wrap: wrap;
    }
    .controls input, .controls select, .controls button {
      padding: 6px 10px;
      font-size: 14px;
      border-radius: 4px;
      border: 1px solid #ccc;
    }
    .controls button {
      cursor: pointer;
      background: #333;
      color: #fff;
      border: none;
    }
    body.dark .controls button { background: #eee; color: #333; }

    .section-title {
      margin-top: 15px;
      font-size: 18px;
      font-weight: bold;
      border-bottom: 2px solid #0073e6;
      padding-bottom: 4px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .section-title select {
      padding: 4px 6px;
      font-size: 14px;
      border-radius: 4px;
    }

    .deal-box {
      border: 1px solid #ddd;
      margin: 10px 0;
      padding: 10px;
      background: #fff;
      border-radius: 6px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
      transition: 0.3s;
    }
    body.dark .deal-box { background: #2b2b2b; border: 1px solid #444; }

    .deal-box h3 { margin: 0; }
    .deal-box p { margin: 5px 0; }

    .deal-box a {
      display: inline-block;
      margin-top: 8px;
      padding: 8px 12px;
      background: #0073e6;
      color: #fff;
      text-decoration: none;
      border-radius: 4px;
      transition: 0.3s;
    }
    .deal-box a:hover { background: #005bb5; }

    .badge {
      display: inline-block;
      margin-left: 8px;
      padding: 2px 6px;
      font-size: 12px;
      border-radius: 4px;
      color: #fff;
    }
    .cheap { background: green; }
    .new { background: orange; }

    .featured { border: 2px solid gold; background: #fffbe6; }
    body.dark .featured { background: #3a3000; border: 2px solid #f2c94c; }

    .pagination {
      display: flex;
      justify-content: center;
      gap: 10px;
      margin-top: 15px;
    }
    .pagination button {
      padding: 6px 12px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      background: #0073e6;
      color: #fff;
    }
    .pagination button:disabled {
      background: #ccc;
      cursor: not-allowed;
    }
  </style>
</head>
<body>
  <div class="controls">
    <input type="text" id="search" placeholder="🔍 Search deals...">
    <select id="categoryFilter">
      <option value="All">All Categories</option>
    </select>
    <select id="sort">
      <option value="default">Sort By</option>
      <option value="low-high">Price: Low → High</option>
      <option value="high-low">Price: High → Low</option>
    </select>
    <button onclick="toggleTheme()">🌙 Dark / ☀️ Light</button>
  </div>

  <div class="section-title">
    🌟 Featured Deals
    <select id="featuredMode">
      <option value="cheapest">Cheapest</option>
      <option value="latest">Latest</option>
    </select>
  </div>
  <div id="featured-container"></div>

  <div class="section-title">🔥 All Deals</div>
  <div id="deals-container">Deals Loading...</div>

  <div class="pagination">
    <button id="prevBtn" onclick="changePage(-1)">⬅ Prev</button>
    <button id="nextBtn" onclick="changePage(1)">Next ➡</button>
  </div>

  <script>
    let allDeals = [];
    let currentPage = 1;
    const dealsPerPage = 10;

    function loadDeals() {
      fetch("deals.json")
        .then(response => response.json())
        .then(data => {
          allDeals = data;
          populateCategories();
          renderFeatured();
          renderDeals();
        })
        .catch(error => {
          document.getElementById("deals-container").innerHTML = "❌ Error loading deals";
          console.error(error);
        });
    }

    function populateCategories() {
      let categories = ["All", ...new Set(allDeals.map(d => d.category))];
      let select = document.getElementById("categoryFilter");
      select.innerHTML = "";
      categories.forEach(cat => {
        let option = document.createElement("option");
        option.value = cat;
        option.textContent = cat;
        select.appendChild(option);
      });
    }

    function renderFeatured() {
      let mode = document.getElementById("featuredMode").value;
      let container = document.getElementById("featured-container");
      container.innerHTML = "";

      let featuredDeals = [];
      if (mode === "cheapest") {
        featuredDeals = [...allDeals].sort((a, b) => a.price - b.price).slice(0, 3);
      } else if (mode === "latest") {
        featuredDeals = [...allDeals].slice(-3).reverse();
      }

      if (featuredDeals.length === 0) {
        container.innerHTML = "<p>⚠️ No featured deals</p>";
      } else {
        featuredDeals.forEach(deal => {
          let badge = mode === "cheapest" ? `<span class="badge cheap">Cheap</span>` : `<span class="badge new">New</span>`;
          let box = document.createElement("div");
          box.className = "deal-box featured";
          box.innerHTML = `
            <h3>⭐ ${deal.title} ${badge}</h3>
            <p><b>Category:</b> ${deal.category}</p>
            <p><b>Price:</b> ₹${deal.price}</p>
            <a href="${deal.url}" target="_blank">Grab Deal</a>
          `;
          container.appendChild(box);
        });
      }
    }

    function renderDeals() {
      let container = document.getElementById("deals-container");
      container.innerHTML = "";

      let searchText = document.getElementById("search").value.toLowerCase();
      let selectedCategory = document.getElementById("categoryFilter").value;
      let sortOption = document.getElementById("sort").value;

      let filteredDeals = allDeals.filter(deal => {
        let matchesSearch = deal.title.toLowerCase().includes(searchText);
        let matchesCategory = selectedCategory === "All" || deal.category === selectedCategory;
        return matchesSearch && matchesCategory;
      });

      if (sortOption === "low-high") {
        filteredDeals.sort((a, b) => a.price - b.price);
      } else if (sortOption === "high-low") {
        filteredDeals.sort((a, b) => b.price - a.price);
      }

      let totalPages = Math.ceil(filteredDeals.length / dealsPerPage);
      if (currentPage > totalPages) currentPage = totalPages;
      if (currentPage < 1) currentPage = 1;

      let start = (currentPage - 1) * dealsPerPage;
      let end = start + dealsPerPage;
      let paginatedDeals = filteredDeals.slice(start, end);

      if (paginatedDeals.length === 0) {
        container.innerHTML = "<p>⚠️ No deals found</p>";
      } else {
        paginatedDeals.forEach(deal => {
          let box = document.createElement("div");
          box.className = "deal-box";
          box.innerHTML = `
            <h3>${deal.title}</h3>
            <p><b>Category:</b> ${deal.category}</p>
            <p><b>Price:</b> ₹${deal.price}</p>
            <a href="${deal.url}" target="_blank">Grab Deal</a>
          `;
          container.appendChild(box);
        });
      }

      document.getElementById("prevBtn").disabled = currentPage === 1;
      document.getElementById("nextBtn").disabled = currentPage === totalPages || totalPages === 0;
    }

    function changePage(direction) {
      currentPage += direction;
      renderDeals();
    }

    document.getElementById("search").addEventListener("input", () => { currentPage = 1; renderDeals(); });
    document.getElementById("categoryFilter").addEventListener("change", () => { currentPage = 1; renderDeals(); });
    document.getElementById("sort").addEventListener("change", () => { currentPage = 1; renderDeals(); });
    document.getElementById("featuredMode").addEventListener("change", renderFeatured);

    function toggleTheme() {
      document.body.classList.toggle("dark");
    }

    setInterval(loadDeals, 300000); // auto-refresh every 5 min
    loadDeals();
  </script>
</body>
</html>
