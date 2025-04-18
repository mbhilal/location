<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Tracking Link Generator</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link
    rel="stylesheet"
    href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.15.3/css/all.min.css"
  />
  <link
    href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap"
    rel="stylesheet"
  />
  <style>
    body {
      font-family: 'Inter', sans-serif;
    }
  </style>
</head>
<body class="bg-gray-50 min-h-screen flex flex-col">
  <header class="bg-white shadow">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex items-center justify-between">
      <h1 class="text-2xl font-semibold text-gray-900 flex items-center">
        <i class="fas fa-map-marker-alt text-indigo-600 mr-2"></i> Link Tracker
      </h1>
    </div>
  </header>

  <main class="flex-grow max-w-4xl mx-auto px-4 sm:px-6 lg:px-8 py-8 w-full">
    <section class="bg-white rounded-lg shadow p-6">
      <h2 class="text-xl font-semibold text-gray-800 mb-4">Create a Tracking Link</h2>
      <form id="linkForm" class="space-y-4">
        <div>
          <label for="targetUrl" class="block text-gray-700 font-medium mb-1">Target URL</label>
          <input
            type="url"
            id="targetUrl"
            name="targetUrl"
            placeholder="https://example.com"
            required
            class="w-full border border-gray-300 rounded-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          />
        </div>
        <div>
          <button
            type="submit"
            class="inline-flex items-center px-4 py-2 bg-indigo-600 text-white font-semibold rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500"
          >
            <i class="fas fa-link mr-2"></i> Generate Tracking Link
          </button>
        </div>
      </form>

      <div id="result" class="mt-6 hidden">
        <label for="trackingLink" class="block text-gray-700 font-medium mb-1">Your Tracking Link</label>
        <div class="flex items-center space-x-2">
          <input
            type="text"
            id="trackingLink"
            readonly
            class="flex-grow border border-gray-300 rounded-md px-3 py-2 bg-gray-100 text-gray-700"
          />
          <button
            id="copyBtn"
            class="inline-flex items-center px-3 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500"
            title="Copy to clipboard"
          >
            <i class="fas fa-copy"></i>
          </button>
        </div>
      </div>
    </section>

    <section class="bg-white rounded-lg shadow p-6 mt-10">
      <h2 class="text-xl font-semibold text-gray-800 mb-4">Tracked Visits</h2>
      <div class="overflow-x-auto">
        <table class="min-w-full divide-y divide-gray-200">
          <thead class="bg-gray-50">
            <tr>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Timestamp</th>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">IP Address</th>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">City</th>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Region</th>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Country</th>
              <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">User Agent</th>
            </tr>
          </thead>
          <tbody id="visitsTableBody" class="bg-white divide-y divide-gray-200 max-h-96 overflow-y-auto block w-full">
            <!-- Rows will be inserted here -->
          </tbody>
        </table>
      </div>
    </section>
  </main>

  <footer class="bg-white shadow mt-12">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 text-center text-gray-500 text-sm">
      &copy; 2024 Link Tracker. All rights reserved.
    </div>
  </footer>

  <script>
    // Utility function to generate a random ID for tracking links
    function generateId(length = 8) {
      const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
      let result = '';
      for (let i = 0; i < length; i++) {
        result += chars.charAt(Math.floor(Math.random() * chars.length));
      }
      return result;
    }

    // Save and load tracking links and visits from localStorage
    function saveTrackingLinks(links) {
      localStorage.setItem('trackingLinks', JSON.stringify(links));
    }
    function loadTrackingLinks() {
      const links = localStorage.getItem('trackingLinks');
      return links ? JSON.parse(links) : {};
    }

    function saveVisits(visits) {
      localStorage.setItem('trackingVisits', JSON.stringify(visits));
    }
    function loadVisits() {
      const visits = localStorage.getItem('trackingVisits');
      return visits ? JSON.parse(visits) : [];
    }

    // Render visits table
    function renderVisits() {
      const visits = loadVisits();
      const tbody = document.getElementById('visitsTableBody');
      tbody.innerHTML = '';
      if (visits.length === 0) {
        const tr = document.createElement('tr');
        tr.className = 'block w-full text-center text-gray-500 py-4';
        const td = document.createElement('td');
        td.colSpan = 6;
        td.textContent = 'No visits tracked yet.';
        tr.appendChild(td);
        tbody.appendChild(tr);
        return;
      }
      visits.forEach((visit) => {
        const tr = document.createElement('tr');
        tr.className = 'block w-full flex flex-col sm:table-row sm:flex-row sm:items-center';
        tr.innerHTML = `
          <td class="px-4 py-2 text-sm text-gray-700 break-words">${new Date(visit.timestamp).toLocaleString()}</td>
          <td class="px-4 py-2 text-sm text-gray-700 break-words">${visit.ip || 'N/A'}</td>
          <td class="px-4 py-2 text-sm text-gray-700 break-words">${visit.city || 'N/A'}</td>
          <td class="px-4 py-2 text-sm text-gray-700 break-words">${visit.region || 'N/A'}</td>
          <td class="px-4 py-2 text-sm text-gray-700 break-words">${visit.country || 'N/A'}</td>
          <td class="px-4 py-2 text-sm text-gray-700 break-words max-w-xs truncate" title="${visit.userAgent || ''}">${visit.userAgent || 'N/A'}</td>
        `;
        tbody.appendChild(tr);
      });
    }

    // Copy tracking link to clipboard
    document.getElementById('copyBtn').addEventListener('click', () => {
      const input = document.getElementById('trackingLink');
      input.select();
      input.setSelectionRange(0, 99999);
      navigator.clipboard.writeText(input.value).then(() => {
        alert('Tracking link copied to clipboard!');
      });
    });

    // Handle form submission to generate tracking link
    document.getElementById('linkForm').addEventListener('submit', (e) => {
      e.preventDefault();
      const targetUrl = e.target.targetUrl.value.trim();
      if (!targetUrl) return;

      const trackingLinks = loadTrackingLinks();
      // Generate unique ID
      let id;
      do {
        id = generateId(10);
      } while (trackingLinks[id]);

      trackingLinks[id] = targetUrl;
      saveTrackingLinks(trackingLinks);

      // Show the tracking link
      const resultDiv = document.getElementById('result');
      const trackingLinkInput = document.getElementById('trackingLink');
      const baseUrl = window.location.origin + window.location.pathname;
      trackingLinkInput.value = `${baseUrl}?track=${id}`;
      resultDiv.classList.remove('hidden');
    });

    // On page load, check if URL has ?track=ID param
    async function handleTracking() {
      const urlParams = new URLSearchParams(window.location.search);
      const trackId = urlParams.get('track');
      if (!trackId) return;

      const trackingLinks = loadTrackingLinks();
      const targetUrl = trackingLinks[trackId];
      if (!targetUrl) {
        // Invalid tracking ID, just do nothing or show error
        return;
      }

      // Fetch visitor IP and location info from a free API
      // Using https://ipapi.co/json/ for demo (no API key required, limited usage)
      let visitorInfo = {
        ip: 'Unknown',
        city: 'Unknown',
        region: 'Unknown',
        country: 'Unknown',
        userAgent: navigator.userAgent,
        timestamp: Date.now(),
      };

      try {
        const response = await fetch('https://ipapi.co/json/');
        if (response.ok) {
          const data = await response.json();
          visitorInfo.ip = data.ip || visitorInfo.ip;
          visitorInfo.city = data.city || visitorInfo.city;
          visitorInfo.region = data.region || visitorInfo.region;
          visitorInfo.country = data.country_name || visitorInfo.country;
        }
      } catch (error) {
        // Fail silently, keep unknown info
      }

      // Save visit info
      const visits = loadVisits();
      visits.push(visitorInfo);
      saveVisits(visits);

      // Redirect to target URL after a short delay to allow data saving
      setTimeout(() => {
        window.location.href = targetUrl;
      }, 500);
    }

    // Initialize
    renderVisits();
    handleTracking();
  </script>
</body>
</html>        
